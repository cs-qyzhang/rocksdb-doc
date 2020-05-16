---
title: "编译"
date: 2020-05-15
lastmod: 2020-05-15
authors: ["张丘洋"]
---

It is possible to carry out many customizations *without* editing any code in the `themes/academic/` folder, making it easier to update the framework in the future.

## Custom theme

Both the **colour theme** and **font set** can be customized.

To **customize a color theme**:

1. Copy a theme such as `themes/academic/data/themes/minimal.toml` to `data/themes/my_theme.toml` (at the root of your site, **not** in `themes/academic/`), creating the `data/themes/` folders if they do not already exist. Note: avoid using spaces in filenames.
2. Adjust the colors within your theme file using [HTML color codes](https://htmlcolorcodes.com)
3. Tell Academic to use your new theme by setting `theme = "my_theme"` in `config/_default/params.toml`

### Custom font

To **customize the font set**:

1. Copy a font set such as `themes/academic/data/fonts/minimal.toml` to `data/fonts/my_font_set.toml` (at the root of your site, **not** in `themes/academic/`), creating the `data/fonts/` folders if they do not already exist. Note: avoid using spaces in filenames.
2. Adjust the font family, choosing from the library of [Google Fonts](https://fonts.google.com) if you wish - refer to the Google Font guide below
3. Tell Academic to use your new font set by setting `font = "my_font_set"` in `config/_default/params.toml`

To select a free web-font available from Google Fonts:

1. Visit [Google Fonts](https://fonts.google.com)
2. Click `+` on the fonts you wish to use
3. Open the `Families Selected` dialog box at the bottom
4. Under **Embed Font**, copy the part of the URL in bold and paste it as the `google_fonts` option in your new font theme
    - For example, if Google gives you `<link href="https://fonts.googleapis.com/css?family=B612+Mono|Open+Sans&display=swap" rel="stylesheet">`, set `google_fonts = "B612+Mono|Open+Sans"`
5. Under **Specify in CSS** in Google Font's dialog, copy the font name and paste it as one of the fonts in your font theme
    - For example, given `font-family: 'B612 Mono', monospace;`, copy `B612 Mono`

### Change font size

You can modify the font size all the way from extra small to extra large using the `font_size` option in `config/_default/params.toml`.

### Example

An example of a custom theme in action can be found in the [repo for this site](https://github.com/sourcethemes/academic-www/blob/master/data/themes/academic.toml) - refer to `data/themes/academic.toml` and the value of `theme` in `params.toml`.

### Share your theme

If you create your own theme, consider giving it a unique name and *sharing* your new theme or font set with the [community](http://discuss.gohugo.io/) or following the guide on our  page so that it can be considered for curation.

## Website icon

Are you looking to customize the desktop and mobile icon which appears in your site's Progressive Web App (PWA) and web browser tab?

Save your icon as a square 512x512 pixel image named `icon.png` and place the image in your root `assets/images/` folder, creating the `assets` and `images` folders if they don't already exist.

Note that the icon might also be referred to as a _favicon_.

{{% alert note %}}
**For versions of Academic prior to v4.7:**

Save your desktop and mobile icons as square PNG images named `icon-32.png` (32x32 pixels), `icon-192.png` (192x192 pixels), and `icon-512.png` (512x512 pixels) respectively. Place them in your root `static/img/` folder.
{{% /alert %}}

## Analytics

If you wish to use [Google Analytics](https://analytics.google.com) or [Google Tag Manager](https://tagmanager.google.com), include your associated tracking code (e.g. `UA-12345678-9`):

```toml
############################
## Marketing
############################
[marketing]
  google_analytics = ""
  google_tag_manager = ""
```

GA only tracks users on your live _production_ website. Hugo's default environments are `development` with the `hugo serve` command and `production` with the `hugo` command. You can explicitly set the environment to production with `HUGO_ENV` or Hugo's `--environment` [option](https://gohugo.io/commands/hugo_env/#readout). If you deploy with Netlify, check that your Netlify settings in `netlify.toml` are similar to the [latest Netlify settings in the _Academic Kickstart_ template](https://github.com/sourcethemes/academic-kickstart/blob/master/netlify.toml).

We recommend testing that your analytics are working as expected. If they are not, check that your production environment is as described above and check that your URL in your GA admin panel matches your site's URL. Furthermore, GA provide a troubleshooting guide to help with any issues.

If you choose to use GA via Google Tag Manager instead (`google_tag_manager`), it will override the direct GA implementation (`google_analytics`) to prevent tracking each user twice.

To integrate other third party analytics services, refer to the **Add Scripts** section.

## Comments

**Academic supports both the extremely popular [Disqus](https://disqus.com) commenting platform and the [Commento](https://commento.io/) fast, Markdown supported, privacy-focused commenting platform.

To configure commenting, open `config/_default/params.toml` in your text editor and scroll down to edit the Comments section:

```toml
# Comments.
[comments]
  # Comment provider:
  #   0: Disabled
  #   1: Disqus (https://disqus.com)
  #   2: Commento (https://commento.io)
  engine = 0

  # Which page types are commentable?
  commentable = {page = true, post = true, docs = true}

  # Configuration of Disqus.
  [comments.disqus]
    shortname = ""  # Paste the shortname from your Disqus dashboard.
    show_count = true  # Show comment count in page header? (true/false)

  # Configuration of Commento.
  [comments.commento]
    # If self-hosting Commento, enter its URL here (e.g. "https://commento.?.com"), otherwise leave empty.
    url = ""
```

## Add scripts (JS)

To add a **third party script**, create a file named `custom_js.html` in a `layouts/partials/` folder at the root of your website (not in the `themes` folder). Any HTML code added to this file will be included within your website's footer.

Whereas for your own **local scripts**, let's say that you have a JS file named `custom.js`. We can place your file in  `assets/js/` (create the folders within your website root if they don't exist). Then open your `config/_default/params.toml` and set `plugins_js = ["custom"]`, where *custom* is your JS filename without its extension.

## Page Sharer

The *Page Sharer* enables you to **grow your audience** by engaging your visitors to share pages on your site with others. Visitors can share your pages via **email, Twitter, Facebook, LinkedIn, WhatsApp, Weibo, and many more!**

To enable the Page Sharer, set `sharing` to `true` in your `config/_default/params.toml`. Alternatively, it can be enabled or disabled on a per page basis using `share` in your page front matter.

To customize the Page Sharer, edit `data/page_sharer.toml`. If you don't already have this file in your site, update to Academic **v4.4+** (if necessary) and copy the `themes/academic/data/page_sharer.toml` file to `data/page_sharer.toml`.

### Select your social networks

To enable or disable a button, set its `enable` option to `true` or `false`.

### Reorder your social networks

To reorder buttons, move the `[[buttons]]` instances to your preferred order.

### Add a new social network

We've already included some of the most popular networks, but further networks can easily be added. To add a button for a new social network, duplicate a `[[buttons]]` instance and edit its properties. , and give it a `title` and a one-word, lowercase `id`. Once you have the sharing URL for the new social network, paste it into the `url` field, substituting the title of the page to share with `{title}` and page URL with `{url}`.

## Custom head

To add custom **metadata** or scripts to your website's `<head>`, create a file named `custom_head.html` in a `layouts/partials/` folder at the root of your website (not in the `themes` folder) and add your HTML code.

This approach can also be used for adding third party scripts with an *async* or *defer* attribute to the site head.

**NOTE: Prior to v4.4.0-dev, `head_custom.html` is used rather than `custom_head.html`.**

## Permalinks

*Permalinks*, or *permanent links*, are URLs to individual pages and posts on your website. They are permanent web addresses which can be used to link to your content. Using Hugo's *permalinks* option these can be easily customized. For example, the blog post URL can be changed to the form *yourURL/2016/05/01/my-post-slug* by adding the following near the end of your `config/_default/config.toml`:

    [permalinks]
        post = "/:year/:month/:day/:slug"

Where `:slug` defaults to the filename of the post, excluding the file extension. However, slug may be overridden on a per post basis if desired, simply by setting `slug = "my-short-post-title"` in your post preamble.

**Example 2:** let's consider changing the URL path of posts from `post/` to `blog/`. First, add the following parameters to your `config/_default/config.toml`:
```
[permalinks]
    post = "/blog/:slug"
```
Then add `aliases = ["/blog/"]` to your post archive page at `post/_index.md` so that it can be accessed from the */blog/* URL.

## Customize style (CSS)

To personalize Academic, you can **choose a colour theme and font set** in `config/_default/params.toml`.

For further personalization, you can [**create your own colour theme and font theme**](#custom-theme).

If advanced style customization is required, **CSS code** can be written to override or enhance the existing styles:

1. Create the `assets/scss/` folder if it doesn't exist
2. Create a file named `custom.scss`
3. Add your custom CSS code to the file you created and re-run Hugo to view changes

**Note: prior to v4.6, custom CSS can be added using the `plugins_css` option in `params.toml`.**

## Override a template

Hugo uses a [template lookup](https://gohugo.io/templates/lookup-order/) which enables you to override any of Academic's files without directly changing them. This can make it easier to update Academic in the future since you do not modify any of Academic's files directly.

To override a template in the theme, you simply copy the file you are interested in from `themes/academic/` and paste it in your site folder using a similar path. To understand how this works, you should familiarize yourself with template lookup. Finally, when you update Academic, remember to compare your version of the file against Academic's one, in case you need to propagate any changes across.

For example, say we wish to add some HTML code to the navigation bar. Let's copy the relevant file `themes/academic/layouts/partials/navbar.html` to `layouts/partials/navbar.html` (at the root of your site, **not** in `themes/academic/`), creating the `layouts/partials/` folders if they do not already exist. Now you can add the HTML code you want to your copy of the file, which will override Academic's version.

## Date Format

Academic uses the following constants to format dates and times:

| Type     | Options |
|----------|--------------------------------------------------------------------------------------------------|
| Year     | `06`   `2006` |
| Month    | `01`   `1`   `Jan`   `January` |
| Day      | `02`   `2`   `_2` <span style="font-size:90%;">(width two, right justified)</span> |
| Weekday  | `Mon`   `Monday` |
| Hours    | `03`   `3`   `15` |
| Minutes  | `04`   `4` |
| Seconds  | `05`   `5` |
| AM/PM    | `PM`   `pm` |
| Timezone | `MST` |
| Offset   | `-0700`   `-07`   `-07:00`   `Z0700`   `Z07:00` |

To modify the format, edit `date_format` and `time_format` in `config/_default/params.toml` using the above constants. For example, some common formats are:

| Format                            | Type              |
|-----------------------------------|-------------------|
| `January 2, 2006`                 | Date              |
| `01/02/06`                        |                   |
| `Jan-02-06`                       |                   |
| `Mon, 02 Jan 2006`                |                   |
| `Monday, 02 Jan 2006`             |                   |
| `15:04`                           | Time              |
| `3:04 PM`                         |                   |
| `15:04 MST`                       |                   |

## Customizing Taxonomy Pages

Academic makes available taxonomies such as tags and categories for organizing content.

If you need to display custom page elements, such as custom title or header image, for each taxonomy term, you can create a page for that term at `/content/<TAXONOMY>/<TERM>/_index.md` and  to its .
