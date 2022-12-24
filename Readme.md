## https://reactrouter.com/en/main/start/tutorial

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

# loading data

URL segments, layouts, and data are more often than not coupled (tripled?) together. We can see it in this app already:

URL Segment Component Data
/ <Root> list of contacts
contacts/:id <Contact> individual contact

Because of this natural coupling, React Router has data conventions to get data into your route components easily.

There are two APIs we'll be using to load data, loader and useLoaderData. First we'll create and export a loader function in the root module, then we'll hook it up to the route. Finally, we'll access and render the data.

That's it! React Router will now automatically keep that data in sync with your UI. We don't have any data yet, so you're probably getting a blank list like this:

# Data Write + Html Forms

We'll create our first contact in a second, but first let's talk about HTML.

React Router emulates HTML Form navigation as the data mutation primitive, according to web development before the JavaScript cambrian explosion. It gives you the UX capabilities of client rendered apps with the simplicity of the "old school" web model.

While unfamiliar to some web developers, HTML forms actually cause a navigation in the browser, just like clicking a link. The only difference is in the request: links can only change the URL while forms can also change the request method (GET vs POST) and the request body (POST form data).

Without client side routing, the browser will serialize the form's data automatically and send it to the server as the request body for POST, and as URLSearchParams for GET. React Router does the same thing, except instead of sending the request to the server, it uses client side routing and sends it to a route action.

We can test this out by clicking the "New" button in our app. The app should blow up because the Vite server isn't configured to handle a POST request (it sends a 404, though it should probably be a 405 🤷).

# Creating contacts

Instead of sending that POST to the Vite server to create a new contact, let's use client side routing instead.

The createContact method just creates an empty contact with no name or data or anything. But it does still create a record, promise!

🧐 Wait a sec ... How did the sidebar update? Where did we call the action? Where's the code to refetch the data? Where are useState, onSubmit and useEffect?!

This is where the "old school web" programming model shows up. As we discussed earlier, <Form> prevents the browser from sending the request to the server and sends it to your route action instead. In web semantics, a POST usually means some data is changing. By convention, React Router uses this as a hint to automatically revalidate the data on the page after the action finishes. That means all of your useLoaderData hooks update and the UI stays in sync with your data automatically! Pretty cool.

# Synchronizing URLs to Form State

There are a couple of UX issues here that we can take care of quickly.

If you click back after a search, the form field still has the value you entered even though the list is no longer filtered.
If you refresh the page after searching, the form field no longer has the value in it, even though the list is filtered
In other words, the URL and our form state are out of sync.

👉 Return q from your loader and set it as the search field default value
