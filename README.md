# Parks Theme Setup

A Drupal recipe for the **Acquia Drupal 12 Site Building** course. Applied by
trainees at the end of Learning Module 1.

It does five things:

1. Installs **Parks Theme** and makes it the default front-end theme
2. Enables **UI Patterns**, which is what makes the theme's seven Single
   Directory Components placeable in Layout Builder
3. Places the theme's 11 default blocks
4. Turns on **Media** and **Media Library**, and applies all five core media
   type recipes so every media type is available
5. Leaves the site ready for Module 6

## Media types

Each core recipe creates exactly one media type, so all five are applied:

| Media type | From |
|---|---|
| Image | `core/recipes/image_media_type` |
| Document | `core/recipes/document_media_type` |
| Remote video | `core/recipes/remote_video_media_type` |
| Audio | `core/recipes/audio_media_type` |
| Local video | `core/recipes/local_video_media_type` |

`image_media_type` also installs `media_library`, `path` and `views`, and
`media_library` depends on `media`, so Media arrives transitively. Both are
still named explicitly in the recipe so its intent does not depend on that.

> **Course note.** Learning Module 3.3 has trainees apply
> `document_media_type` and `remote_video_media_type` themselves. Because this
> recipe now applies them at the end of Module 1, that exercise will find the
> work already done. Drop those two lines from `recipes:` if you want 3.3 back.

## Install and apply

### DDEV

```bash
ddev composer config repositories.parks_theme vcs https://github.com/imrodmartin/parks_theme
ddev composer config repositories.parks_theme_setup vcs https://github.com/imrodmartin/parks_theme_setup
ddev composer require imrodmartin/parks_theme_setup:^1.0
ddev drush cache:rebuild
ddev exec bash -c "cd web && ../vendor/bin/dr recipe:apply ../recipes/parks_theme_setup"
```

### Acquia Cloud IDE

```bash
composer config repositories.parks_theme vcs https://github.com/imrodmartin/parks_theme
composer config repositories.parks_theme_setup vcs https://github.com/imrodmartin/parks_theme_setup
composer require imrodmartin/parks_theme_setup:^1.0
drush cache:rebuild
cd docroot && ../vendor/bin/dr recipe:apply ../recipes/parks_theme_setup
```

Use `web` instead of `docroot` if that is your docroot.

## Do not skip the cache rebuild

Drupal caches its list of available modules and themes. Composer has just put a
new theme on disk, and until the cache is rebuilt Drupal does not know it
exists. Skip that step and the recipe fails with:

```
Validation errors were found in recipe.yml:
 - [install][4]: "parks_theme" is not a known module or theme.
```

That message means the cache is stale, not that the theme is missing.

## What to expect

`composer require` installs the theme to `themes/custom/parks_theme`, the recipe
to `recipes/parks_theme_setup`, and `drupal/ui_patterns` to `modules/contrib`.

Drupal then **unpacks** the recipe: its dependencies are hoisted into your root
`composer.json` and the recipe itself is removed from your requirements. The
recipe files stay in `recipes/` so they can be applied. This is normal, and it
is why `imrodmartin/parks_theme_setup` will not appear in your `composer.json`
afterwards.

## Verify

```bash
drush config:get system.theme default          # parks_theme
drush pm:list --status=enabled --filter=patterns
drush pm:list --status=enabled --filter=media
drush php:eval 'print implode(", ", array_keys(\Drupal::entityTypeManager()->getStorage("media_type")->loadMultiple()));'
```

The last command should print all five: `audio, document, image, remote_video, video`.

Then visit **Structure > Block layout**. The Parks Theme tab should show blocks
already placed in Header, Primary menu, Breadcrumb, Content and Footer.

## Recipes cannot be un-applied

There is no uninstall and no rollback. Snapshot your database first if you want
a way back.

## Licence

GPL-2.0-or-later
