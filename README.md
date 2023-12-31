# A blog website

#### Home page

![Alt text](blog_homePage.png)



#### About page

![Alt text](aboutPage.png)


#### Creat new blog
![Alt text](newBlog.png)


#### Details page
![Alt text](details.png)

#### Mongo_db

![Alt text](mongodb.png)


```js
// connect to mongodb
mongoose
  .connect(dbURI)
  // listen for requests
  .then((result) => app.listen(3000))
  .catch((err) => console.log(err));

```

```js
//middleware & static files
app.use(express.static("public"));
app.use(express.urlencoded({ extended: true }));
app.use(morgan("dev"));
app.use((req, res, next) => {
  res.locals.path = req.path;
  next();
});

```

```js

// routes
app.get("/", (req, res) => {
  res.redirect("/blogs");
});

app.get("/about", (req, res) => {
  res.render("about", { title: "About" });
});

// blog routes
app.use("/blogs",blogRoutes);

// 404 page
app.use((req, res) => {
  res.status(404).render("404", { title: "404" });
});

```

```js
// Controllers
const blog_index = (req, res) => {
  Blog.find()
    .sort({ createdAt: -1 })
    .then((result) => {
      res.render("blogs/index", { blogs: result, title: "All blogs" });
    })
    .catch((err) => {
      console.log(err);
    });
};

const blog_details = (req, res) => {
  const id = req.params.id;
  Blog.findById(id)
    .then((result) => {
      res.render("blogs/details", { blog: result, title: "Blog Details" });
    })
    .catch((err) => {
      res.status().render("404", {title: "Blog not found"})
    });
};

const blog_create_get = (req, res) => {
  res.render("blogs/create", { title: "Create a new blog" });
};

const blog_create_post = (req, res) => {
  const blog = new Blog(req.body);
  blog
    .save()
    .then((result) => {
      res.redirect("/blogs");
    })
    .catch((err) => {
      console.log(err);
    });
};

const blog_delete = (req, res) => {
  const id = req.params.id;
  Blog.findByIdAndDelete(id)
    .then((result) => {
      res.json({ redirect: "/blogs" });
    })
    .catch((err) => console.log(err));
};

module.exports = {
  blog_index,
  blog_details,
  blog_create_get,
  blog_create_post,
  blog_delete,
};
```

```js
// Models
const mongoose = require("mongoose");
const Schema = mongoose.Schema;

const blogSchema = new Schema(
  {
    title: {
      type: String,
      required: true,
    },
    snippet: {
      type: String,
      required: true,
    },
    body: {
      type: String,
      required: true,
    },
  },
  { timestamps: true }
);

const Blog = mongoose.model("Blog", blogSchema);

module.exports = Blog;

```

```js
// Views
<html lang="en">
  <%- include("../partials/head.ejs") %>

  <body>
    <%- include("../partials/nav.ejs") %>

    <div class="blogs content">
      <h2>All Blogs</h2>

      <% if (blogs.length > 0) { %> <% blogs.forEach(blog => { %>
      <a href="/blogs/<%= blog._id %>" class="single">
        <h3 class="title"><%= blog.title %></h3>
        <p class="snippet"><%= blog.snippet %></p>
      </a>

      <% }) %> <% } else { %>
      <p>There are no blogs to display...</p>
      <% } %>
    </div>

    <%- include("../partials/footer.ejs") %>
  </body>
</html>
```