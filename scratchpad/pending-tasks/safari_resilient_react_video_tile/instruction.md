Safari restricts auto-playing media elements that receive new source tracks while the browser tab is in the background. If a remote user unmutes while the local user is in another tab, traditional `srcObject` reassignment triggers Safari's security blocks, causing the video stream to freeze when returning to the tab.

You need to create a `ParticipantTile.tsx` React component that renders a remote participant's video stream while entirely avoiding Safari's background auto-play block.

**Constraints:**
- You MUST use the `useVideoTrack(sessionId)` hook to retrieve the participant's video state.
- You MUST rely on the `persistentTrack` property (e.g., `videoState.persistentTrack`) and attach it to the `<video>` element's `srcObject` inside a `useEffect`. Do NOT use the deprecated `track` property.
- The component MUST display a fallback UI `<div>` with the text "Camera Off" when `videoState.state` is not equal to `'playable'`.