---
title: Conditional rendering based on the agent platform in Phoenix
date: "2024-11-12"
description: "Three ways to conditionally render HTML components based on the agent platform in Phoenix."
---

For [Tuist Previews](https://docs.tuist.dev/en/guides/share/previews#previews), we needed to conditionally render a button – on desktop, users would be able to run a Preview using the [Tuist macOS app](https://docs.tuist.dev/en/guides/share/previews#tuist-macos-app). On mobile devices, users should be able to install the app directly.

Let's take a look at three ways we could achieve that in Phoenix applications.

## Setting up development environment on macOS

To be able to test changes, we have generally two options, both useful. Note the following steps are for macOS-specific.

The easier and quicker way to test our changes is to override the user agent in the browser. In Firefox, you can go to `about:config` and update the `general.useragent.override` option to the user agent you want to test. For an iOS device, an example user agent could be `Mozilla/5.0 (iPhone; CPU iPhone OS 13_5_1 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/13.1.1 Mobile/15E148 Safari/604.1`.

The second option is to use view the page directly on the mobile device – that's not always needed, but recommended for more complex features. To test your changes on a real device using your locally running server, you first need to grab your IP address. On macOS, you can copy your address from `System Preferences > Network > Wi-Fi details`.

Additionally, you need to update your `config/dev.exs` file to run at `0.0.0.0` instead of `127.0.0.1` which is the default value:
```elixir
config :tuist, TuistWeb.Endpoint,
  http: [ip: {0, 0, 0, 0}, port: 8080],
  # Instead of default of `http: [ip: {127, 0, 0, 1}, port: 8080],`
```

Now, you can head to `your-ip-address:8080` on your mobile device to view the locally running server.

## Pure JavaScript

What we started with was a static HTML component and a pure JavaScript solution in the `<script>` tag:
```html
<script>
  function iOS() {
    return [
      'iPad Simulator',
      'iPhone Simulator',
      'iPod Simulator',
      'iPad',
      'iPhone',
      'iPod'
    ].includes(navigator.platform)
  }

  function hideElementsByClass(className) {
    var elements = document.querySelectorAll('.' + className);
    elements.forEach(function(element) {
      element.style.display = 'none';
    });
  }

  if (iOS()) {
      hideElementsByClass('desktop-button');
  } else {
    hideElementsByClass('mobile-button');
  }
</script>
```

In short, we read the platform from the user agent and then hide the appropriate element based on the platform. This was a quick and easy solution which worked well!

However, at some point, we had the need to start using a `LiveView`. Hiding element in `<script>` in a `LiveView` _won't work_ as it's not given that the elements are not present when the script runs. We could hide the elements after a given timeout, but that leads to layout shifts, which is not what we wanted.

## Using Phoenix Hooks

Ok, when `<script>` doesn't cut it, we need to turn to Phoenix JS primitives. This actually felt like a great use case for [Phoenix hooks](https://hexdocs.pm/phoenix_live_view/js-interop.html#client-hooks-via-phx-hook). We could define a `PlatformSpecificVisibility` to conditionally render the HTML components – and because the hook would be run when the component mounts, we no longer would have to depend on the timeouts.

The hook could be implemented as such:
```javascript
function iOS() {
  return [
    "iPad Simulator",
    "iPhone Simulator",
    "iPod Simulator",
    "iPad",
    "iPhone",
    "iPod",
  ].includes(navigator.platform);
}

Hooks.PlatformSpecificVisibility = {
  visibleOn() {
    return this.el.dataset.visibleOn;
  },
  mounted() {
    switch (this.visibleOn()) {
      case "ios":
        if (!iOS()) {
          this.el.style.display = "none";
        }
        break;
      case "desktop":
        if (iOS()) {
          this.el.style.display = "none";
        }
        break;
      default:
        break;
    }
  },
};
```

Note the `visibileOn` property – we can use the data property to specify at the component level on which environments it should be visible.

The HTML components could then be implemented as such:
```html
<button
  id="install-button"
  phx-hook="PlatformSpecificVisibility"
  data-visible-on="ios"
/>
<button
  id="run-button"
  phx-hook="PlatformSpecificVisibility"
  data-visible-on="desktop"
/>
```

And this works great! But since the JavaScript kicks in only _after_ the component has mounted, we still get a flash of the content. We could hide the buttons by default to make that less intrusive – but still not great.

## Reading the user agent at the `LiveView` level

Use JavaScript only when you need it. _Really_. And in this case, it turns out we don't to use JavaScript at all! Instead, we can read the user agent from the `socket` when initially mounting the component in the `LiveView` using the [ua_parser](https://hexdocs.pm/ua_parser/readme.html) library:
```elixir
def mount(
  _params,
  _session,
  socket
) do
  {
    :ok,
    socket
    |> assign(
      :is_ios,
      case UAParser.parse(get_connect_info(socket, :user_agent)) do
        %UAParser.UA{os: %UAParser.OperatingSystem{family: family}} -> family == "iOS"
        _ -> false
      end
    )
  }
end
```

And then we can conditionally render the buttons based on the `is_ios` value:
```html
<%= if @is_ios do %>
  <button id="install-button" />
<% else %>
  <button id="run-button" />
<% end %>
```

As mentioned in this [post](https://fly.io/phoenix-files/pass-user-agent-info-to-your-liveview/), we need to pass `:user_agent` to `connect_info` in the `endpoint.ex` file, so the user agent is always available and we never see any flash of content:
```elixir
- socket "/live", Phoenix.LiveView.Socket, websocket: [connect_info: [session: @session_options]]
+ socket "/live", Phoenix.LiveView.Socket, websocket: [connect_info: [:user_agent, session: @session_options]]
```

And that's it! We now have a solution that doesn't rely on JavaScript and doesn't flash any content 🎉
