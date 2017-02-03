
<p align="center"><em>The example above was created with Slate. Check it out at <a href="https://lord.github.io/slate">lord.github.io/slate</a>.</em></p>

Features
------------

Getting Started with Slate
------------------------------

### Prerequisites

You're going to need:

 - **Linux or OS X** — Windows may work, but is unsupported.
 - **Ruby, version 2.2.5 or newer**
 - **Bundler** — If Ruby is already installed, but the `bundle` command doesn't work, just run `gem install bundler` in a terminal.

### Editing site

1. Add yourself as a contributor to this project.
2. Clone to your hard drive with `git clone git@github.com:yummytech/DeveloperHub.git`
3. `cd DeveloperHub`
4. Initialize and start Slate.

```shell
# either run this to run locally
bundle install
bundle exec middleman server
```
You can now see the docs at http://localhost:4567.

To deploy to the active site

Commit your changes to the markdown source: git commit -a -m "Update index.md"
Push the markdown source changes to Github: git push
```shell 
# to deploy to active site
./deploy.sh
```

Edit the index.html.md file to change the content.
[Editing Slate markdown](https://github.com/lord/slate/wiki/Markdown-Syntax)

### Troubleshooting

This currently only works with old versions of ruby due to a bug in the middleman package. 
```shell
# to downgrade ruby
rvm install 2.2.5
```

If bundler is not installed properly
```shell 
gem install bundler
```

If middleman is not installed properly
```shell 
gem install middleman
```
