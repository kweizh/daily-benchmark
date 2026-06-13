In large calls, subscribing to every participant's audio and video can overwhelm the client's network. However, attempting to manually subscribe or unsubscribe to specific tracks throws an exception (`setSubscribedTracks is not supported...`) if Daily's state model expects automatic handling.

You need to write a JavaScript function using `@daily-co/daily-js` that creates a headless call object specifically configured for manual, audio-only selective subscriptions.

**Constraints:**
- You MUST configure the call object factory options to set `subscribeToTracksAutomatically: false` upon creation.
- You MUST configure the factory options to disable the local camera by setting `videoSource: false`.
- You MUST include a subsequent function call that manually updates the subscription profile of a specific target `participantId` to receive their `audio` track while keeping their `video` track unsubscribed.