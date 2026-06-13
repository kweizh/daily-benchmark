### 1. Library Overview

*   **Description**: Daily (daily.co) is a developer platform that provides real-time video, audio, and AI-assisted calling APIs built on WebRTC. It manages global WebRTC infrastructure, signaling, media negotiation, TURN servers, and network traversal, allowing developers to embed video calls or build custom headless audio/video applications.
*   **Ecosystem Role**: Daily fits into the real-time communication (RTC) layer of modern software stacks. It provides two main modes of integration:
    1.  **Daily Prebuilt**: A ready-to-use, fully featured video chat UI embedded in an iframe.
    2.  **Call Object (Headless)**: A headless, low-level SDK that provides raw media tracks (`MediaStreamTrack`) and call state, allowing developers to build completely custom user interfaces.
    It also integrates tightly with server-side AI pipelines (via `daily-python` and Pipecat) and transcription services (like Deepgram).
*   **Project Setup**:
    To initialize a new project with Daily, developers use standard package managers. There is no official interactive TUI wizard; setup is entirely declarative and non-interactive.

    **Web/React Setup**:
    ```bash
    # Create a new React application
    npx create-react-app daily-video-app --template typescript
    cd daily-video-app

    # Install Daily Client SDK, React Hooks library, and peer dependency Jotai
    npm install @daily-co/daily-js@0.81.0 @daily-co/daily-react@0.21.3 jotai
    ```

    **Python Server-Side Setup**:
    ```bash
    # Set up Python virtual environment
    python3 -m venv venv
    source venv/bin/activate

    # Install Daily Python Client SDK
    pip install daily-python==0.20.0
    ```

---

### 2. Core Primitives & APIs

Daily's architecture relies on several core primitives across client-side, server-side, and layout engines.

| Concept / API | SDK / Interface | Description | Reference Link |
| :--- | :--- | :--- | :--- |
| **`DailyIframe` Factory** | Client JS | Entry point for creating Prebuilt frames or headless Call Objects. | [DailyIframe Docs](https://docs.daily.co/reference/daily-js) |
| **`DailyCall` Instance** | Client JS | The active call session controller returned by the factory. | [Daily Call Client Docs](https://docs.daily.co/reference/daily-js/daily-call-client) |
| **`DailyProvider`** | React | React Context provider that exposes the call object state to all child hooks. | [DailyProvider Docs](https://docs.daily.co/reference/daily-react/daily-provider) |
| **`useParticipantIds`** | React | React hook to retrieve a filtered, sorted list of participant session IDs. | [useParticipantIds Docs](https://docs.daily.co/reference/daily-react/use-participant-ids) |
| **`useVideoTrack` / `useAudioTrack`** | React | React hooks to retrieve the state and raw `MediaStreamTrack` for a participant. | [useMediaTrack Docs](https://docs.daily.co/reference/daily-react/use-media-track) |
| **`CallClient`** | Python SDK | Core client class for joining calls and capturing raw media streams on the server. | [CallClient Docs](https://docs.daily.co/reference/daily-python/concepts/call-client) |
| **`EventHandler`** | Python SDK | Subclassable class to handle server-side call and participant events. | [EventHandler Docs](https://docs.daily.co/reference/daily-python/api/event-handler) |
| **Rooms API** | REST API | Endpoints to programmatically create, delete, and configure WebRTC rooms. | [REST API Rooms Docs](https://docs.daily.co/docs/rest-api/index.md) |
| **Meeting Tokens API** | REST API | Endpoints to generate access tokens for private rooms or assign owner privileges. | [Meeting Tokens Docs](https://docs.daily.co/docs/guides/privacy-and-security/meeting-tokens) |
| **Video Component System (VCS)** | Layout Engine | React-based layout engine used to build custom cloud recording and streaming layouts. | [VCS Core Concepts Docs](https://docs.daily.co/docs/vcs/core-concepts/index.md) |

#### Key API Implementations & Code Snippets

##### A. Client-Side React Integration (Headless Call Object Mode)
This example demonstrates how to set up a custom video call interface using `@daily-co/daily-react` hooks. It showcases the recommended modern pattern of using `persistentTrack` and `tracks.video.state` rather than deprecated properties.

```tsx
// App.tsx
import React, { useRef } from 'react';
import { DailyProvider, useCallObject, useParticipantIds, useVideoTrack, DailyAudio } from '@daily-co/daily-react';

export default function App() {
  // useCallObject automatically handles creation and cleanup, avoiding StrictMode duplicate issues.
  const callObject = useCallObject({
    subscribeToTracksAutomatically: true,
  });

  const handleJoin = async () => {
    if (!callObject) return;
    await callObject.join({ url: 'https://your-domain.daily.co/room-name' });
  };

  const handleLeave = async () => {
    if (!callObject) return;
    await callObject.leave();
  };

  return (
    <DailyProvider callObject={callObject}>
      <div style={{ padding: 20 }}>
        <button onClick={handleJoin}>Join Call</button>
        <button onClick={handleLeave}>Leave Call</button>
        <ParticipantGrid />
        {/* DailyAudio automatically handles rendering all remote audio elements */}
        <DailyAudio />
      </div>
    </DailyProvider>
  );
}

// ParticipantGrid.tsx
function ParticipantGrid() {
  const remoteParticipantIds = useParticipantIds({ filter: 'remote' });
  const localParticipantIds = useParticipantIds({ filter: 'local' });
  const allIds = [...localParticipantIds, ...remoteParticipantIds];

  return (
    <div style={{ display: 'grid', gridTemplateColumns: 'repeat(auto-fill, minmax(200px, 1fr))', gap: 15, marginTop: 20 }}>
      {allIds.map((id) => (
        <ParticipantTile key={id} sessionId={id} />
      ))}
    </div>
  );
}

// ParticipantTile.tsx
function ParticipantTile({ sessionId }: { sessionId: string }) {
  const videoState = useVideoTrack(sessionId);
  const videoEl = useRef<HTMLVideoElement>(null);

  // Safely attach the persistentTrack to the video element
  React.useEffect(() => {
    if (videoEl.current && videoState.persistentTrack) {
      videoEl.current.srcObject = new MediaStreamTrack([videoState.persistentTrack]);
    }
  }, [videoState.persistentTrack]);

  const isPlayable = videoState.state === 'playable';

  return (
    <div style={{ border: '1px solid #ccc', borderRadius: 8, padding: 10, position: 'relative' }}>
      <p style={{ margin: '0 0 10px 0' }}>ID: {sessionId.slice(0, 8)}</p>
      <video
        ref={videoEl}
        autoPlay
        playsInline
        muted={sessionId === 'local'}
        style={{
          width: '100%',
          height: '150px',
          backgroundColor: '#000',
          display: isPlayable ? 'block' : 'none',
        }}
      />
      {!isPlayable && (
        <div style={{ height: 150, backgroundColor: '#333', color: '#fff', display: 'flex', alignItems: 'center', justifyContent: 'center' }}>
          Camera Off ({videoState.state})
        </div>
      )}
    </div>
  );
}
```
*Other SDK equivalents*: In vanilla JavaScript (`daily-js`), you achieve the same behavior by listening to `call.on('participant-updated')` and manually querying `call.participants()`.

##### B. Server-Side REST API (Room Creation & Token Generation)
This Node.js snippet shows how to create a private room and generate an authorized meeting token with owner privileges.

```javascript
const DAILY_API_KEY = process.env.DAILY_API_KEY;

// 1. Create a private room with knocking (waiting room) enabled
async function createPrivateRoom(roomName) {
  const response = await fetch('https://api.daily.co/v1/rooms', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${DAILY_API_KEY}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      name: roomName,
      privacy: 'private',
      properties: {
        enable_knocking: true,
        enable_chat: true,
        exp: Math.floor(Date.now() / 1000) + 3600, // Expire room in 1 hour
      },
    }),
  });

  const room = await response.json();
  return room; // Returns { url, name, privacy, ... }
}

// 2. Create an owner meeting token for the private room
async function createOwnerToken(roomName, userName) {
  const response = await fetch('https://api.daily.co/v1/meeting-tokens', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${DAILY_API_KEY}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      properties: {
        room_name: roomName,
        is_owner: true,
        user_name: userName,
        exp: Math.floor(Date.now() / 1000) + 3600,
      },
    }),
  });

  const tokenData = await response.json();
  return tokenData.token; // Returns JWT token
}
```

##### C. Server-Side Python Client SDK (Media Pipeline Integration)
This Python script uses `daily-python` to join a call, disable automatic video subscriptions to save bandwidth, and register an audio renderer to capture raw PCM16 audio.

```python
import sys
import threading
from daily import Daily, CallClient, EventHandler

# Initialize Daily SDK globally
Daily.init()

class AudioCaptureHandler(EventHandler):
    def __init__(self):
        super().__init__()
        self.joined = threading.Event()

    def on_participant_joined(self, participant):
        print(f"Participant joined: {participant['info']['userName']}")

    def on_participant_left(self, participant, reason):
        print(f"Participant left: {participant['id']} due to {reason}")

def on_audio_data(participant_id, audio_data):
    # audio_data contains raw PCM16 bytes
    print(f"Received {len(audio_data)} bytes of audio from {participant_id}")

def main():
    room_url = "https://your-domain.daily.co/room-name"
    meeting_token = "your-optional-token"

    handler = AudioCaptureHandler()
    client = CallClient(event_handler=handler)

    # Disable automatic subscriptions to manually select audio-only
    client.update_subscription_profiles({
        "base": {
            "camera": "unsubscribed",
            "microphone": "subscribed"
        }
    })

    # Simple join completion callback
    def on_join(data, error):
        if error:
            print(f"Failed to join: {error}")
            sys.exit(1)
        print("Joined successfully!")
        handler.joined.set()

    client.join(room_url, meeting_token=meeting_token, completion=on_join)
    handler.joined.wait(timeout=10)

    # Set up an audio renderer for all remote participants
    # This renders audio to mono PCM16 at 16,000 Hz
    # Note: Replace '*' with a specific participant ID if needed
    client.set_participant_audio_renderer(
        "*", 
        callback=on_audio_data, 
        sample_rate=16000, 
        num_channels=1
    )

    try:
        # Keep process alive to receive audio
        threading.Event().wait(timeout=30)
    except KeyboardInterrupt:
        pass
    finally:
        # CRITICAL CLEANUP: Prevent thread and media pipeline leaks
        print("Cleaning up Daily client...")
        client.leave()
        client.release()

if __name__ == "__main__":
    main()
```

---

### 3. Real-World Use Cases & Templates

1.  **[custom-video-daily-react-hooks](https://github.com/daily-demos/custom-video-daily-react-hooks)**: The flagship React template demonstrating how to build a custom multi-party video calling interface. It showcases haircheck device selection, active speaker detection, custom layout grids, and text chat.
2.  **[virtual-class-demo](https://github.com/daily-demos/virtual-class-demo)**: A Next.js-based classroom orchestration template showcasing advanced user roles (teacher vs. student), knock-to-join permissions, and pre-authorization flows.
3.  **[prebuilt-transcription](https://github.com/daily-demos/prebuilt-transcription)**: Next.js demo demonstrating how to embed Daily Prebuilt and integrate Deepgram to display live, real-time speech-to-text transcriptions directly inside the UI.
4.  **[daily-bots-web-demo](https://github.com/daily-demos/daily-bots-web-demo)**: A template illustrating how to build real-time voice-first AI agents using Pipecat (Python agent) and RTVI (Real-Time Voice/Video Inference) on top of Daily's WebRTC transport layer.

---

### 4. Developer Friction Points

#### Friction Point 1: React Strict Mode Duplicate Call Instances
*   **Description**: In React development environments, Strict Mode intentionally mounts and unmounts components twice. This often triggers an immediate error when initializing a Daily call instance in a standard `useEffect`.
*   **Symptom**: `Error: Duplicate DailyIframe instances are not allowed` or `Dual call object instances detected`.
*   **Underlying Cause**: Daily's client engine allows only one active `DailyCall` instance per page by default. When React mounts the component a second time before the first async `.destroy()` completes, a duplicate instance is detected.
*   **Resolution**: Developers should use the `@daily-co/daily-react` hook `useCallObject()` or `useCallFrame()` which internally orchestrates singletons and prevents race conditions. Alternatively, developers can pass `allowMultipleCallInstances: true` to the factory options.
*   **Reference**: [Cleaning up the iFrame in React useEffect is impossible #229](https://github.com/daily-co/daily-js/issues/229)

#### Friction Point 2: Calling `setSubscribedTracks` with Automatic Subscriptions Enabled
*   **Description**: Attempting to manually subscribe or unsubscribe from specific participants' audio or video tracks throws a runtime exception.
*   **Symptom**: `Error: setSubscribedTracks is not supported when subscribeToTracksAutomatically is true`.
*   **Underlying Cause**: By default, Daily automatically subscribes the local participant to all incoming streams. If `subscribeToTracksAutomatically` is set to `true`, calling manual subscription methods violates the state model.
*   **Resolution**: Set `subscribeToTracksAutomatically: false` in the factory options of `createCallObject()` or call `setSubscribeToTracksAutomatically(false)` before attempting manual updates.
*   **Reference**: [Best practices to scale large experiences](https://docs.daily.co/guides/scaling-calls/best-practices-to-scale-large-experiences)

#### Friction Point 3: `daily-python` Resource and Thread Leaks
*   **Description**: Running server-side Python bots or media pipelines that continually connect and disconnect can lead to memory depletion or container crashes.
*   **Symptom**: Python processes leak OS and WebRTC worker threads, and memory consumption continuously grows over time.
*   **Underlying Cause**: Creating a `CallClient` launches underlying native C++ WebRTC worker threads. Simply leaving the room with `client.leave()` does not clean up these native threads.
*   **Resolution**: Developers must explicitly call `client.leave()` followed by `client.release()` when destroying the `CallClient` instance to clean up underlying C++ processes.
*   **Reference**: [daily.CallClient thread leak #33](https://github.com/daily-co/daily-python/issues/33)

#### Friction Point 4: Safari Background Tab Auto-Play Block
*   **Description**: In Safari, when a remote participant unmutes their video or audio while the local user's browser tab is in the background, the media elements fail to play.
*   **Symptom**: Video streams freeze or audio remains muted when returning to the tab, with console warnings about play requests being interrupted.
*   **Underlying Cause**: Safari restricts auto-playing media elements that receive new source tracks while the page is in the background. If using the deprecated `track` property, the element's source is reassigned upon unmuting, triggering Safari's security blocks.
*   **Resolution**: Use the `persistentTrack` property on the participant track object (e.g., `tracks.video.persistentTrack`). `persistentTrack` keeps a constant reference to the media stream attached to the DOM element even when muted, avoiding new auto-play triggers when unmuting.
*   **Reference**: [Working with video call participants' media tracks](https://docs.daily.co/blog/working-with-video-call-participants-media-tracks-for-fun-and-profit/)

---

### 5. Evaluation Ideas

1.  **Private Room Setup (Simple)**: Implement a Node.js backend endpoint that creates a private Daily room and returns an owner meeting token.
2.  **Theme Customization (Simple)**: Embed Daily Prebuilt on a web page and programmatically toggle between light and dark themes using custom colors.
3.  **Dynamic Grid Layout (Medium)**: Build a custom React grid that displays remote participants' video using `useVideoTrack` and displays a placeholder when their camera state is not `playable`.
4.  **Audio-Only Streaming (Medium)**: Configure a headless call client to join a room with `videoSource: false` and `subscribeToTracksAutomatically: false`, and manually subscribe only to active speakers' audio.
5.  **Server-Side Audio Capture (Medium)**: Write a Python script using `daily-python` that joins a room, captures raw PCM16 audio from remote participants, and saves it to a local file.
6.  **Knock-to-Join Waiting Room (Complex)**: Implement a complete waiting room flow where owners can view a list of knocking participants and approve or deny them in real-time.
7.  **Spatial Audio Simulation (Complex)**: Disable automatic track subscriptions and dynamically subscribe or unsubscribe to remote participants' audio tracks based on their simulated 2D coordinate distance.
8.  **VCS Layout Live Stream (Complex)**: Trigger a live stream to an RTMP endpoint using a custom VCS layout preset that splits the screen for two active speakers and renders a ticker overlay.

---

### 6. Sources

1.  [Daily Docs Index](https://docs.daily.co/llms.txt) - Structured overview of Daily's entire developer documentation.
2.  [Daily JS Client Reference](https://docs.daily.co/reference/daily-js) - Complete API reference for `daily-js` client capabilities and factory methods.
3.  [Daily React Hooks Reference](https://docs.daily.co/reference/daily-react/daily-provider) - Guide to `DailyProvider` and React integration.
4.  [Daily Python SDK concepts](https://docs.daily.co/reference/daily-python) - Quickstart and installation documentation for the Python SDK.
5.  [Daily REST API Reference](https://docs.daily.co/reference/rest-api/index) - Endpoint specifications for rooms, meeting tokens, and recording.
6.  [VCS Core Concepts](https://docs.daily.co/reference/vcs) - Engine specifications for Video Component System layouts.
7.  [Daily Demos Github Org](https://github.com/daily-demos) - Collection of official full-stack boilerplates and example projects.
8.  [daily-python GitHub Issues](https://github.com/daily-co/daily-python/issues) - Developer discussions regarding thread leaks and callbacks.
9.  [daily-js GitHub Issues](https://github.com/daily-co/daily-js/issues) - Developer discussions regarding React Strict Mode iframe cleanup.