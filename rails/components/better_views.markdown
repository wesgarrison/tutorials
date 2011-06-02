# View Layer Best Practices

The view layer is the most ignored part of our stack. We tend to think that the "real programming" happens at the model layer, the controllers are an inconvenience, and the views are just for designers.

That's just not true. We can add in components and techniques to the Rails stack that make the views more beautiful, functional, and easier to write. 

For this tutorial, we'll make use of a version of the JSBlogger sample application. Check out the repository and begin at the `starter` tag:

`git clone -b better_views git://github.com/jcasimir/jsblogger.git`
`git checkout starter`

## Understanding the View Stack

Before we dive into our views in depth, we need to have a common understanding of the request lifecycle and the view's responsibilities. Here's a typical Rails request:

[TODO: Format and insert my Rails stack graphic]

The view receives data from the controller and prepares output for the user. The preparation step includes both formatting that data and combining it with view templates to generate the finished product. Let's look at templating first.

## Rails Templating with ERB and HAML

The "golden path" in Rails still uses *Embedded Ruby* or ERB. The template files live in `app/views/` and are named after the controller and action they're attached to.

### Reviewing ERB

In ERB we have three main markup elements:

* Plain HTML and text use no markers and just appear plainly on the page
* `<%=` and `%>` wrap Ruby code whose return value will be output in place of the marker
* `<%` and `%>` wrap Ruby code whose return value will *NOT* be output
* `<%-` and `%>` wrap Ruby code whose return value will *NOT* be output and no blank lines will be generated

The last of those, using `<%-`, has become much less significant as modern browsers restructure the DOM into a browsable tree where whitespace doesn't matter.

### Why Hate ERB?

The great thing about the ERB system itself is that it's totally generalized. In Rails applications we use it to create HTML files, but there's no reason we couldn't use ERB to output JavaScript, configuration files, or even form letters. ERB doesn't know anything about the surrounding text, it's just for injecting printing or non-printing Ruby code.

But that's a downside, too. ERB is very general, and a more specialized solution can help reduce our workload.

### Enter HAML

ERB vs. HAML has the fervor of a religious debate in the Rails community. According to surveys, about 46% of the community prefers HAML. Expert teams almost always prefer HAML except in situations where designers need to be completely "plug and play." Long story short, use HAML unless you have a strong reason not to.

### HAML Setup

Open the project's `Gemfile` and add this dependency:

```ruby
gem "haml"
```

Save it and run `bundle`

Once your dependencies are setup, start the server with `rails server`

### HAML Prototype

HAML was developed by Hampton Caitlin as pushback against the "heavy" nature of HTML and, by extension, XML. He started HAML by taking an ERB template and making one assumption: white space should be significant. With that assumption, he started deleting all the characters that could possibly be inferred by the template processor engine.

### A First Refactoring

Start by visiting http://127.0.0.1:3000/ and you'll see the article listing page for JSBlogger. Open the view template in `app/views/articles/index.html.erb`

On line 1 you'll see this H1:

```html
<h1>All Articles</h1>
```

It could have been written like this:

```html
<h1>
  All Articles
</h1>
```

And if we assume that whitespace is significant, the close tag would become unnecessary here. The parser could know that the H1 ends when there's an element at the same indentation level as the opening H1 tag. Cut the closing tag and we have:

```html
<h1
  All Articles
```

With the H1 tag itself, the `>` seem unnecessary. Leaving just `<h1` as the HTML marker could have worked, but Hampton decided that HTML elementes would be created with `%` like this:

```html
%h1
  All Articles
```

Lastly, when an HTML element just has one line of content, we'll conventionally put it on a single line:

```html
%h1 All Articles
```

Flip over to your browser, refresh the index page, and you'll see the plain `%h1 All Articles` output on the page. HAML is loaded, but it isn't parsing the document because it's still named `index.html.erb`

*Rename* the template to `index.html.haml` and refresh your page. Then you should get an error about illegal nesting. In your text editor, move all the lines so they're flush against the left edge, save, and refresh. The template should render without error, but it will look ridiculous.

### Outputting Ruby in HAML

On line 3 you should have:

```html
<p class='flash'><%= flash[:notice] %></p>
```

Given what you've seen from the H1, you would imagine the `<p></p>` becomes `%p`. But what about the Ruby injection?

HAML's approach is to reduce `<%= %>` to just `=`. The benefit is many fewer characters to type, but the cost is that with this syntax a single line must either be plain text or Ruby code -- no mixing.

```html
%p= flash[:notice]
```

Note that the `=` must be touching the `%p`

But what about the class? There are two options. The most robust syntax using the Ruby 1.9 hash style is like this:

```html
%p{class: 'flash'}= flash[:notice]
```

But when we just have a single class, we can also use a CSS-like syntax:

```html
%p.flash= flash[:notice]
```

I'd recommend the last one wherever possible.

### Mixing Plain Text and Ruby

Fix up the new article link yourself. It doesn't have an HTML tag, so just start the line with the `=`

The sidebar has a DIV with content and Ruby injection:

```ruby
<div id="sidebar">
Filter by Tag: <%= tag_links(Tag.all) %>
</div>
```

You'd be tempted to go this direction. Note that the ID is using a CSS-style syntax:

```ruby
%div#sidebar Filter by Tag: = tag_links(Tag.all)
```

But HAML won't recognize the Ruby code there. It'll just output the code as plain text. The traditional solution is to put the plain text and the Ruby on their own lines indented under the DIV:

```ruby
%div#sidebar
  Filter by Tag: 
  = tag_links(Tag.all)
```

Since HAML 3, though, there's an interpolation-like syntax for situations like this where you're mixing plain text and Ruby:

```ruby
%div#sidebar
  Filter by Tag: #{tag_links(Tag.all)}
```

Now it can be pushed up to one line:

```ruby
%div#sidebar Filter by Tag: #{tag_links(Tag.all)}
```

And finally, DIV is considered the "default" HTML tag. If you just use a CSS-style ID or Class with no explicit HTML element, HAML will assume a DIV:

```ruby
#sidebar Filter by Tag: #{tag_links(Tag.all)}
```

### Non-Printing Ruby

Now you're left with the UL. The two challenges here are that you have some non-printing Ruby and elements within other elements.

On the outside, start the UL with `%ul#articles` and the other lines will all be indented two spaces below it. Delete its closing tag.

Next you have the `each` loop which uses non-printing Ruby lines. In HAML, these lines begin with a minus `-` like this:

```ruby
%ul#articles
  - @articles.each do |article|
  <li>
  <%= link_to article.title, article_path(article) %>
  <span class='tag_list'><%= article.tag_list %></span>
  <span class='actions'>
  <%= edit_icon(article) %>
  <%= delete_icon(article) %>
  </span>
  </li>
  - end
```

Refresh your browser and it'll show you an error -- the `end` is unnecessary in HAML. The same whitespace respect that simplifies HTML can simplify our Ruby! Delete the `- end` line, refresh, and your rendering will still break. All those lines formerly inside the `do` / `end` need to be indented two spaces inside the `do` line:

```ruby
%ul#articles
  - @articles.each do |article|
    <li>
    <%= link_to article.title, article_path(article) %>
    <span class='tag_list'><%= article.tag_list %></span>
    <span class='actions'>
    <%= edit_icon(article) %>
    <%= delete_icon(article) %>
    </span>
    </li>
```

Now your rendering should succeed.

### The Child Elements

* Convert the LI element to the HAML syntax, indent the content lines two spaces, and delete the closing tag.
* Change the `<%=` lines to just `=` and remove the closing tags.
* Rebuild the SPAN elements using the HAML style for the tag and the CSS class
* Make sure to indent the edit and delete icons so they live inside the actions SPAN

### Completed Template

```ruby
%h1 All Articles

%p.flash= flash[:notice]

= link_to "New Article", new_article_path, :class => 'new_article'

#sidebar Filter by Tag: #{tag_links(Tag.all)}

%ul#articles
  - @articles.each do |article|
    %li
      = link_to article.title, article_path(article)
      %span.tag_list= article.tag_list
      %span.actions
        = edit_icon(article)
        = delete_icon(article)
```

### Exercises

Try to rebuild the `show.html.erb` into `show.html.haml`. Remember you can push everything over to the left edge, allowing you to refactor it one element at a time. When you're struggling to represent the structure, try separating parts into their own lines, then reduce them down as you see fit.