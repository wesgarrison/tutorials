# Jumpstart Lab Tutorials

## Status

The site can now be used with Jekyll locally or viewed via GitHub pages at http://jumpstartlab.github.com/tutorials/

## Usage

View the tutorials online at http://jumpstartlab.github.com/tutorials/

Run them locally by checking out this repository, then:

```bash
bundle
bundle exec jekyll
```

Open http://localhost:4000/ in your browser.

## Issues

GitHub pages does not yet support the latest version of Jekyll/RedCarpet, so the markdown tutorials using fenced code blocks will not work properly. To view them, run Jekyll locally, open `_config.yml` and change `markdown: rdiscount` to `markdown: redcarpet`.

The materials are in a mix of Textile and Markdown. I'm working on converting everything to Markdown.

The index pages are just placeholders, you need to figure out the URLs for individual tutorials manually.

## Editing

Please make edits to the original markdown/textile and submit pull requests. I'm currently working through the JSMerchant tutorial. You'll likely find `"Jeff's Revision Marker"` in the text which is where I've worked up to so far.