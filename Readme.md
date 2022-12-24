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

# Fix "back" button issue

Now for problem (1), clicking the back button and updating the input. We can bring in useEffect from React to manipulate the form's state in the DOM directly.

Now as you type, the form is submitted automatically!

Note the argument to submit. We're passing in event.currentTarget.form. The currentTarget is the DOM node the event is attached to, and the currentTarget.form is the input's parent form node. The submit function will serialize and submit any form you pass to it.

# Adding Search Spinner

In a production app, it's likely this search will be looking for records in a database that is too large to send all at once and filter client side. That's why this demo has some faked network latency.

Without any loading indicator, the search feels kinda sluggish. Even if we could make our database faster, we'll always have the user's network latency in the way and out of our control. For a better UX, let's add some immediate UI feedback for the search. For this we'll use useNavigation again.

The navigation.location will show up when the app is navigating to a new URL and loading the data for it. It then goes away when there is no pending navigation anymore.

# Managing the History Stack

Now that the form is submitted for every key stroke, if we type the characters "seba" and then delete them with backspace, we end up with 7 new entries in the stack 😂. We definitely don't want this

We only want to replace search results, not the page before we started searching, so we do a quick check if this is the first search or not and then decide to replace.

Each key stroke no longer creates new entries, so the user can click back out of the search results without having to click it 7 times 😅.

# Mutations Without Navigation

So far all of our mutations (the times we change data) have used forms that navigate, creating new entries in the history stack. While these user flows are common, it's equally as common to want to change data without causing a navigation.

For these cases, we have the useFetcher hook. It allows us to communicate with loaders and actions without causing a navigation.

The ★ button on the contact page makes sense for this. We aren't creating or deleting a new record, we don't want to change pages, we simply want to change the data on the page we're looking at.

👉 Change the <Favorite> form to a fetcher form

# Optimistic UI

You probably noticed the app felt kind of unresponsive when we clicked the the favorite button from the last section. Once again, we added some network latency because you're going to have it in the real world!

To give the user some feedback, we could put the star into a loading state with fetcher.state (a lot like navigation.state from before), but we can do something even better this time. We can use a strategy called "optimistic UI"

The fetcher knows the form data being submitted to the action, so it's available to you on fetcher.formData. We'll use that to immediately update the star's state, even though the network hasn't finished. If the update eventually fails, the UI will revert to the real data.

If you click the button now you should see the star immediately change to the new state. Instead of always rendering the actual data, we check if the fetcher has any formData being submitted, if so, we'll use that instead. When the action is done, the fetcher.formData will no longer exist and we're back to using the actual data. So even if you write bugs in your optimistic UI code, it'll eventually go back to the correct state 🥹

# No Found Data

Our root errorElement is catching this unexpected error as we try to render a null contact. Nice the error was properly handled, but we can do better!

Whenever you have an expected error case in a loader or action–like the data not existing–you can throw. The call stack will break, React Router will catch it, and the error path is rendered instead. We won't even try to render a null contact.

Instead of hitting a render error with Cannot read properties of null, we avoid the component completely and render the error path instead, telling the user something more specific.

This keeps your happy paths, happy. Your route elements don't need to concern themselves with error and loading states.

# Pathless Routes

One last thing. The last error page we saw would be better if it rendered inside the root outlet, instead of the whole page. In fact, every error in all of our child routes would be better in the outlet, then the user has more options than hitting refresh.

We'd like it to look like this:

We could add the error element to every one of the child routes but, since it's all the same error page, this isn't recommended.

There's a cleaner way. Routes can be used without a path, which lets them participate in the UI layout without requiring new path segments in the URL. Check it out:

👉 Wrap the child routes in a pathless route

When any errors are thrown in the child routes, our new pathless route will catch it and render, preserving the root route's UI!
