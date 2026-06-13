Daily Prebuilt provides a fully featured video chat UI embedded in an iframe. To seamlessly integrate into modern web applications, the Prebuilt UI needs to react dynamically to the host application's theme changes (e.g., toggling between light and dark mode) without dropping the active call.

You need to write a vanilla JavaScript implementation that instantiates a Daily Prebuilt iframe and provides a function to toggle its visual theme on the fly.

**Constraints:**
- You MUST initialize the iframe using `DailyIframe.createFrame()` attached to a specific DOM element.
- You MUST implement a `toggleTheme()` function that updates the iframe's `theme` property (switching between `'light'` and `'dark'`).
- The theme update MUST be applied programmatically using Daily's configuration update methods without calling `.destroy()` or forcing the user to rejoin the room.