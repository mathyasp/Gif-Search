---
# Searching Tenor
---

So now we have done some templating, so we are ready to build the actual feature of the app we want: a gif Search.

We've setup the template, so now we want to use the Tenor API to get some data.

An **API** stands for **Application Programming Interface**. If a **UI** is a **User Interface**, or how humans use a program, an API is the way another computer program uses a program. APIs are other web servers with their own url endpoints much like the ones you are creating now. Instead of returning HTML templates, however, API endpoints return structured data called **JSON**.

> JSON is a series key/value pairs that can be nested into a hierarchical tree structure, for example:

```
  {
    user: {
      name: "Samuel",
      age: 15,
      hometown: "Iowa City"
      friends: [
        { name: "David" },
        { name: "Sarah" },
        { name: "Kelly" }
      ]
    }
  }
```
> We see in the above example that the `user` object has multiple key/value pairs associated with it. An example being the key `name` with the value `"Samuel"`.

# Updating the Template

It is a **Best Practice** to work from the "outside-in" so from what the user experiences in the views back to the controller logic and database structure. So we can start by making an HTML template (enhanced by Handlebars) ready to display images from the tenor API.

We'll use the Handlebars.js `{{#each}}` iterator to loop over the array of gifs that the Tenor API will return.


> Update `/views/home.handlebars` to include the gifs that the Tenor API returns:

```html
<!-- home.handlebars -->
...
>
{{#each gifs}}
{{!-- set the data-width to whatever you want --}}
{{!-- Note that we're using properties of the gif (id, itemurl, url) to fill in information needed to render the gif --}}
    <div class="tenor-gif-embed" data-postid="{{this.id}}" data-share-method="host" data-width="500px"
      data-aspect-ratio="1.2266666666666666"><a
          href="{{this.itemurl}}"></a>
      <a href="{{this.url}}"></a></div>
    <script type="text/javascript" async src="https://tenor.com/embed.js"></script>
{{/each}}
```

We access each gif's `id`, `itemurl` (gif source), and `url` (link to tenor page with the gif) and we give a `data-width` of `500px` so all the gifs are the same width.

If you reload `home` you won't see anything yet, because `gifs` is undefined. Let's define it using the Tenor API.

# Connecting to Tenor API with `TenorJS`

There are two options for using an API. Either you can use the native Node `http` module to call `http.get()` to make a request to these APIs, or someone might have made a **wrapper** to make communicating with an API easier.

In this example we'll first add the **wrapper** library and use that. Then as a stretch challenge you can try to implement the native node.js `http` module.

Let's use the tenor API wrapper called `tenorjs` that makes interacting with the tenor API much simpler. Here are the [tenor api docs](https://tenor.com/gifapi/documentation) if you want to see what other options you have. Be sure to read through the [tenorjs docs](https://www.npmjs.com/package/tenorjs), as that's what we'll be using to interact with the API.

# Adding the Wrapper Library


> First add `tenorjs` to your project's `package.json` file and `node_modules` folder using this bash command:

```bash
$ npm install tenorjs --save
```

> Follow the instructions listed in the [tenorjs docs](https://www.npmjs.com/package/tenorjs). You'll need to make a free developer account, which you can do [here](https://tenor.com/developer/keyregistration). **Remember to verify your email after making your account before you use your key**

> Next change your `app.js` file to require `tenorjs`, and edit the root route to look like the following, replacing `TENOR_API_KEY` with your actual API Key from your account:

  ```js
  // app.js
  // Require Libraries
  ...
  // Require tenorjs near the top of the file
  const Tenor = require("tenorjs").client({
    // Replace with your own key
    "Key": "TENOR_API_KEY", // https://tenor.com/developer/keyregistration
    "Filter": "high", // "off", "low", "medium", "high", not case sensitive
    "Locale": "en_US", // Your locale here, case-sensitivity depends on input
});

  ...

  // Routes
  app.get('/', (req, res) => {
    // Handle the home page when we haven't queried yet
    term = ""
    if (req.query.term) {
        term = req.query.term
    }
    // Tenor.search.Query("SEARCH KEYWORD HERE", "LIMIT HERE")
    Tenor.Search.Query(term, "10")
        .then(response => {
            // store the gifs we get back from the search
            const gifs = response;
            // pass the gifs as an object into the home page
            res.render('home', { gifs })
        }).catch(console.error);
  })
  ```

## What did we just add?

Let's break down what happened above in a bit more detail:

`const Tenor = require("tenorjs").client({... })`

This is how we'll require the `TenjorJS` API wrapper, which will allow us to interact with the Tenor API! We'll save our reference to it in a `const`, making sure to supply it with required parameters that tell us what GIF filter and locale we're in. Most importantly, we're passing our private API key, which lets Tenor know we're authorized to access its API

> As a best practice, we do _not_ want to store private API keys plainly in our code, as it leaves us vulnerable to having that information stolen. For the purpose of this tutorial, we want you to get familiar with the basics of route logic and simple API calls. In future classes, you will learn how to securely store information like this so you don't deploy private information to the public!

`app.get('/', (req, res) => { ... }`

Here we define our route! A route defines the URL that will make a certain page display. The `/` route is the root route. So the our GIF site we're building should display at http://localhost:3000/

Now navigate to `/` and see your gifs. You should see 10 gifs, as that's the limit we gave `tenorjs`

`term = ""
if (req.query.term) {
    term = req.query.term
}`

Did we pass in a search query term? We can check `req.query.term` to see if one exists, and if so, set it so we can search for it.

`Tenor.Search.Query(term, "10")
    .then(response => {
        const gifs = response;
        res.render('home', { gifs })
    }).catch(console.error);
})`

Finally, here is our API call! We use `TenorJS` to call Tenor's Search API through `Search.Query`. We pass along our search term, and that we want the first 10 results. the `then` function is used to describe what should happen once the call to the API is completed. Once Tenor responds with our 10 GIFs, then we want to set that response to a `const` (`gifs`) that we pass along to our page. We then call `res.render` to render our `home` page, passing along the `gifs` as an object for us to use and display on the home page. And if anything goes wrong during our API call, then that's what `catch` is for! We'll log the error the console, and keep the site from blowing up in an error.

## Whew, ok I feel better now. Let's try it!

When you refresh your root route with nothing in the form, why do you still see GIFs?

> The answer is because the Tenor API is smart enough to respond with trending GIFs if no search query term is supplied. Nice huh!?

Try adding a query term to your url to see an actual search: e.g. `?term=kittens`

# Now Commit

```bash
$ git add .
$ git commit -m 'added working gif search'
$ git push
```

Great work! You've successfully **integrated a simple API into your project!**

# Connecting to the Tenor API with `http` (optional stretch challenge)

Now that we've setup receiving the `tenorjs` package. As a strech challenge you can implement the query to the API without a library using Node's native `http` module, and deal with the stream of data coming from that http endpoint.

> Update your `app.js` root route to use the `http` to query the [Tenor API](https://tenor.com/gifapi/documentation)

<!-- -->

> Node's `http` module uses a data type called a **stream** to receive data as it "streams" into your server. That is why we have to add all the data into the `body` variable and wait until it is done streaming with the `response.on('end')` function. Weird right!? Node does this to take advantage of its unblocking I/O nature. What does that mean?

Still work?

This called **refactoring** code. Developers refactor code to make code more **elegant** meaning more economical, more efficient, and more easy to read.
