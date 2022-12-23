# Create a router

This first route is what we often call the "root route" since the rest of our routes will render inside of it. It will serve as the root layout of the UI, we'll have nested layouts as we get farther along.

# Change path

If changed the path to "tests", then will need use "/tests", to reach the same page.ss

# Add contacts route

However, it's not inside of our root layout 😠

# nested routes

We want the contact component to render inside of the <Root> layout like this.
You'll now see the root layout again but a blank page on the right.
We need to tell the root route where we want it to render its child routes. We do that with <Outlet>.

# Client side route

You may or may not have noticed, but when we click the links in the sidebar, the browser is doing a full document request for the next URL instead of using React Router.

Client side routing allows our app to update the URL without requesting another document from the server. Instead, the app can immediately render new UI. Let's make it happen with <Link>.

You can open the network tab in the browser devtools to see that it's not requesting documents anymore
