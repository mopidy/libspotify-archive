# libspotify FAQ

This content was retrieved from
https://web.archive.org/web/20170913122345/https://developer.spotify.com/technologies/libspotify/faq/
on 2018-05-27.

---

Here we answer some common questions about the Libspotify SDK.

**Important!** If you are building applications for iOS or Android, please use
the iOS SDK or Android SDK. LibSpotify and CocoaLibSpotify is considered
deprecated for these platforms.


## libspotify objects

### Why are functions named like `sp_session_inbox_create()`?

Functions ending in `_create()` create a new object inside libspotify rather
than creating some entity on the Spotify servers. These `_create()` functions
leave a reference for the API user to release when the user has no more
interest in the object. *It is important to note that failing to release the
entity will result in memory leaks.*

### I don't get any metadata callback when I create an album or artist link but I do get a callback if I do the same for a track

Creating an `sp_album` or `sp_artist` object will not automatically populate
the object with data. To get information about an artist or album you need to
perform a browse request for libspotify to populate its internal datamodel.
Once the browse request has succeded all `sp_album` and `sp_artist` object that
can be derived from the response will be filled with data.

The `sp_album` and `sp_artist` objects behave more like placeholder objects
that contain minimal information about the artist and album. For example, it's
not possible to get a list of all tracks on an album from just the `sp_album`
object itself.

This diagram describes the internal relationship between the metadata objects:

```
                +----------------+
                | sp_albumbrowse |-----------+ (List of tracks)
                +----------------+           |
                       |                     |
                       V                     V
                 +----------+          +----------+
+--------------->| sp_album |<---------| sp_track |<-----------------+
|                +----------+          +----------+                  |
|                      |                     |                       |
|                      V                     | (Additional artists)  |
|                +-----------+               |                       |
|                | sp_artist |<--------------+                       |
|                +-----------+                                       |
| (List of all         ^                                             |
|  albums for          |                                             |
|  artist)     +-----------------+  (List of all tracks for artist)  |
+--------------| sp_artistbrowse |-----------------------------------+
               +-----------------+
```


## Threading and concurrency

### According to documentation, libspotify is not thread safe. What does this mean?

It means that you can not invoke any libspotify API function while another API
is invoked in a different thread. This includes *all* of the functions, even
`sp_session_process_events()`. In other words, having a thread that just spins
and invokes `sp_session_process_events()` when needed (either due to that the
notification callback fires or a timeout happens), while another thread invokes
API functions, is *not* supported. If you look in our examples you will see
that we never do this ourselves.

### But libspotify creates threads itself!

Yes, but only for serving requests internally and for those requests libspotify
provides adequate locking by itself. In particular it spawns a network thread
that handles all of the communication with Spotify's servers and is also
responsible for decoding music, etc.

### Why is it like this?

libspotify is comprised of two things. The Spotify Core ("Core") which has the
libspotify API layer "on-top" of it. Core is used in all Spotify's native
applications (Desktop client, Mobile clients, etc) and for those GUI centric
applications the `sp_session_process_events()` handing is part of the normal GUI
main thread.

### According to documentation, some callbacks are called from “internal session thread”. What does that mean?

It means that those are not called from inside `sp_session_process_events()`
but rather the network thread as described above.

As of this writing those callbacks are:

- `session.notify_main_thread`
- `session.end_of_track`
- `session.music_delivery`
- `session.start_playback`
- `session.stop_playback`
- `session.get_audio_buffer_stats`

There might be more in the future and the API documentation is the
authoritative information about what callbacks are invoked from what threads.

It's *very* important that these functions *NEVER* block for an extended period
of time. Doing so may cause the API to be unresponsive because you would in
fact block and stop all communication with the Spotify servers. Worst case the
session could be disconnected.

You should not call any other libspotify API methods from within these
callbacks. Even if you use a global lock around all API calls as acquiring this
global lock can take an extended amount of time this would effectively mean
that you block this thread and risk audio dropouts or network disconnects, etc

The most common error is to block in the `music_delivery` callback and wait for
the audio output FIFO/queue/device/etc to have enough available samples to
satisfy the request. This is *not* how it should be done, instead if there are
not enough available buffers the function should just return 0 and libspotify
will retry in a short while again with the same samples.

### What is the the `notify_main_thread` callback? Why is it needed?

This callback is invoked from any thread when the main thread need to awaken to
process events and in turn deliver callbacks to your application.

It's important to remember that the main thread can also invoke this callback.
The reason for this is that it simplifies the internal design. In particular
this happens when cached browse results are loaded. Therefore, the notification
mechanism must be prepared for this. This means that if you plan to take a
lock, it must not already be held when calling `sp_session_process_events()`.

### So how should I write my integration to libspotify?

There are basically two options.

Staying on one thread or adding locking. Which one is better or preferred is
mostly up to the implementer but here is a general sketch:

#### Staying on one thread

This means that you would encode your requests (browse, search, play track,
etc) using some kind of message and post this for processing onto the same main
thread that keeps doing `sp_session_process_events()`

One example of this method is the spshell example. See `spshell_posix.c` and
`spshell_win32.c` in particular. It's very simplistic as it just posts the
command line as typed from the prompt into the main loop and let the main loop
parse the line. But it should be enough to illustrate the general idea.

This approach is probably the best if you already have a design where the
libspotify integration receives messages from an external process or a
different thread. In case you receive the messages from an external process
(via a socket or similar) you probably would need to rewrite the
`notify_main_thread` to use a `pipe(2)` or similar and use `poll(2)` or
`select(2)` to control the main loop. To process messages from a different
thread, you can stick to pthread condition variables much like the spshell
example.

#### Add your own locks

You can add a global lock around all function calls to libspotify. This would
also include `sp_session_process_events()`.

For code inside callbacks that are called indirectly by
`sp_session_process_events()` there is no need to worry about this global lock
as the global lock is already acquired by the current thread outside of
`sp_session_process_events()`. See the question about "callbacks are called from
'internal session thread'" to see what callbacks that are not invoked from
within `sp_session_process_events()` and thus, where you would need additional
locking.

Also beware that during `sp_session_process_events()` data which is owned by
libspotify may change. An example:

Thread A (normal main loop):

``` c
pthread_mutex_lock(&global_lock);
do {
  sp_session_process_events(g_session, &next_timeout);
} while (next_timeout == 0);
pthread_mutex_unlock(&global_lock);
```

Thread B:

``` c
const char *name;

pthread_mutex_lock(&global_lock);
name = sp_playlist_name(playlist);
pthread_mutex_unlock(&global_lock);
printf("The playlist is called: %sn", name);   // <- might crash!
```

After thread B releases `global_lock` thread A might execute and process events
from server that renames the playlist and then the memory holding the name
might be freed resulting in a use of freed memory.

Instead, the proper implementation is:

Thread B:

``` c
const char *name;

pthread_mutex_lock(&global_lock);
name = sp_playlist_name(playlist);
printf("The playlist is called: %sn", name);
pthread_mutex_unlock(&global_lock);
```


## Playlists

### What is an `sp_playlistcontainer`?

A playlistcontainer is a list of playlist. Those are currently obtained from
two sources:

`sp_session_playlistcontainer()` to get the currently logged in users rootlist
and `sp_session_publishedcontainer_for_user_create()`

The latter leaves a reference for the API user to release.

### How/When is a playlist loaded?

Playlist start to load data from Spotify servers as soon as the object
(`sp_playlist`) representing the playlist is created in libspotify. Playlist
objects can be created as a direct result of an API call or implicitly created
because they are part of a playlistcontainer.

To get notifications about changes to the playlist, register callbacks using
`sp_playlist_add_callbacks()`. Before registering the callbacks the API user
should query the current playlist status (enumerate tracks already added, get
playlist name, etc) Once the attributes changes or tracks are added, removed,
moved the relevant callbacks will fire.

When the playlist is loaded it only means that the track objects `sp_track`
representing the tracks has been loaded into memory. You might still need to
wait for metadata about the tracks. Although, if everything was in on disk
cache and nothing have changed all metadata would have been loaded by now. For
more info, see "How/When do I get metadata for tracks on playlists?"

The application should continue and listen to the playlist callbacks as long as
the playlist is displayed and update the view presented to the user so it
reflects changes in the playlist.

Playlist processing and resulting callbacks will only take place during
`sp_session_process_events()` so it's safe to extract the list of tracks and
then register the callbacks (or vice verse) without risking a race condition.

### Why isn't the track working even though `sp_track_is_loaded()` returns true?

Even though `sp_track_is_loaded()` returns true, that doesn't necessarily mean
that the track is ok. One must really check with `sp_track_error()` that the
status is `SP_ERROR_OK`, before calling any other `sp_track_*()` functions.

### How/When do I get metadata for tracks on playlists?

Contrary to browse and search results the playlist load metadata
asynchronously. When metadata for one or more track in a playlist is loaded the
`playlist_metadata_changed` callback will fire. This callback only fires when a
track that is in the playlist transition from non-loaded to loaded state.

In other words, if metadata for all tracks is already loaded the callback will
not fire. The API user is expected to try to extract the metadata (or at least
keep track of which tracks have metadata) when the track is added to the list
(either via the `tracks_added` callback or via the `sp_playlist_track()`
enumeration)

Note that the `playlist_metadata_callback` may also fire spuriously. The reason
for this is that track metadata is loaded in batch requests from Spotify's
servers and if a lot of tracks was requested in one batch it's cheaper for the
libspotify to invoke the callback on all playlists than having to sort out (an
O(n^2) operation) which tracks are in which playlist.

If the same track appears in multiple playlists (in this case the `sp_track`
object would be the same as it's reference counted) the callback would fire for
both playlists.

### What's the correct way to handle folders in the rootlist?

Folders are represented using begin / end markers. One options is to just
ignore those markers. This will present the user will a flattered list (without
any folder). This is certainly not the recommended way and might serve as a
stop gap during development.
