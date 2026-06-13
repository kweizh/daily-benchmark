Daily's Python SDK (`daily-python`) allows servers to join calls and process raw media. However, because it spins up native C++ WebRTC worker threads, improperly shutting down the client will leak OS threads and memory, eventually crashing the container.

You need to write a Python script that joins a Daily room, captures raw audio from all remote participants, and cleanly shuts down after 30 seconds without leaking threads.

**Constraints:**
- You MUST use `daily.CallClient()` and register an audio renderer using `client.set_participant_audio_renderer()` set to 16,000 Hz, mono (1 channel).
- The script MUST disable automatic track subscriptions and manually subscribe ONLY to the `microphone` track to save bandwidth.
- You MUST include a `finally` block that explicitly calls both `client.leave()` and `client.release()` to guarantee native C++ thread cleanup.