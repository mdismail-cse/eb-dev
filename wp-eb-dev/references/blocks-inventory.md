# Essential Blocks — Blocks Inventory

> **TL;DR:** Lookup table for all FREE blocks: slug, PHP class, JS dir,
> static-vs-dynamic, commit count. Read when you need to find a block by
> name, identify its files, or pick a reference block to mirror. For pro
> blocks, see `pro-architecture.md` §5.

All 60+ blocks with slug, PHP class, JS path, and context. Blocks are
registered in `includes/blocks.php`. Inner/child blocks (form fields, tab,
accordion-item, column, price) are not standalone — they register as child
blocks with a `parent` constraint.

## Top-level blocks

Path columns omit the leading `essential-blocks/` and `src/blocks/` to
save space. Registry key is the snake_case key in `includes/blocks.php`.

### Content blocks

| Registry key         | WP name (dashes)        | PHP class          | JS dir                  | Type    | Notes                                              |
|----------------------|-------------------------|--------------------|-------------------------|---------|----------------------------------------------------|
| `accordion`          | `accordion`             | `Accordion`        | `accordion/`            | static  | Popular. Nests `accordion-item` child blocks. 170+ commits |
| `accordion_item`     | `accordion-item`        | `AccordionItem`    | `accordion-item/`       | static  | Child of accordion. Heavy edit churn (26+ commits) |
| `advanced_heading`   | `advanced-heading`      | `AdvancedHeading`  | `advanced-heading/`     | static  | Popular. Heading with separator + gradients       |
| `button`             | `button`                | `Button`           | `button/`               | static  | Second-most-touched block (217 commits)           |
| `dual_button`        | `dual-button`           | `DualButton`       | `dual-button/`          | static  | Two buttons side-by-side with "or" connector      |
| `feature_list`       | `feature-list`          | `FeatureList`      | `feature-list/`         | static  | Icon + text rows                                    |
| `flipbox`            | `flipbox`               | `FlipBox`          | `flipbox/`              | static  | Front/back flip card                                |
| `infobox`            | `infobox`               | `InfoBox`          | `infobox/`              | static  | Icon/image + heading + description + button         |
| `notice`             | `notice`                | `Notice`           | `notice/`               | static  | Info/warning/success alerts                         |
| `pricing_table`      | `pricing-table`         | `PricingTable`     | `pricing-table/`        | static  | Price card with features list + CTA                |
| `team_member`        | `team-member`           | `TeamMember`       | `team-member/`          | static  | Photo + name + role + socials                       |
| `testimonial`        | `testimonial`           | `Testimonial`      | `testimonial/`          | static  | Quote + author                                       |
| `toggle_content`     | `toggle-content`        | `ToggleContent`    | `toggle-content/`       | static  | Show/hide content toggle (129+ edit-file commits)   |
| `wrapper`            | `wrapper`               | `Wrapper`          | `wrapper/`              | static  | Generic section wrapper (container)                 |

### Creative / media blocks

| Registry key         | WP name                 | PHP class          | JS dir                  | Type    | Notes                                              |
|----------------------|-------------------------|--------------------|-------------------------|---------|----------------------------------------------------|
| `countdown`          | `countdown`             | `CountDown`        | `countdown/`            | static  | Date/time countdown timer                           |
| `image_comparison`   | `image-comparison`      | `ImageComparison`  | `image-comparison/`     | static  | Before/after slider                                  |
| `image_gallery`      | `image-gallery`         | `ImageGallery`     | `image-gallery/`        | static  | Top JS-edit hotspot (61+ commits)                   |
| `instagram_feed`     | `instagram-feed`        | `InstagramFeed`    | `instagram-feed/`       | dynamic | IG API integration                                   |
| `interactive_promo`  | `interactive-promo`     | `InteractivePromo` | `interactive-promo/`    | static  | Hover-reveal promo card                              |
| `lottie_animation`   | `lottie-animation`      | `LottieAnimation`  | `lottie-animation/`     | static  | Lottie JSON player (38+ commits)                    |
| `nft_gallery`        | `nft-gallery`           | `NftGallery`       | `nft-gallery/`          | dynamic | NFT display via API                                  |
| `number_counter`     | `number-counter`        | `NumberCounter`    | `number-counter/`       | static  | Animated number counter                              |
| `openverse`          | `openverse`             | `Openverse`        | `openverse/`            | dynamic | Openverse image search integration                   |
| `parallax_slider`    | `parallax-slider`       | `ParallaxSlider`   | `parallax-slider/`      | static  | Parallax scrolling slider                            |
| `popup`              | `popup`                 | `PopUp`            | `popup/`                | dynamic | Modal popup block                                    |
| `progress_bar`       | `progress-bar`          | `ProgressBar`      | `progress-bar/`         | static  | Animated progress indicator                          |
| `shape_divider`      | `shape-divider`         | `ShapeDivider`     | `shape-divider/`        | static  | SVG shape divider section break                      |
| `slider`             | `slider`                | `Slider`           | `slider/`               | static  | Third-most-touched block (180+ commits)             |
| `social`             | `social`                | `Social`           | `social/`               | static  | Social profile links                                  |
| `social_share`       | `social-share`          | `SocialShare`      | `social-share/`         | static  | Share buttons                                         |
| `typing_text`        | `typing-text`           | `TypingText`       | `typing-text/`          | static  | Typewriter effect                                     |

### Query / dynamic content

| Registry key         | WP name                 | PHP class          | JS dir                  | Type    | Notes                                              |
|----------------------|-------------------------|--------------------|-------------------------|---------|----------------------------------------------------|
| `post_grid`          | `post-grid`             | `PostGrid`         | `post-grid/`            | dynamic | Flagship query block. `PostBlock` base             |
| `post_carousel`      | `post-carousel`         | `PostCarousel`     | `post-carousel/`        | dynamic | Carousel variant                                     |
| `post_meta`          | `post-meta`             | `PostMeta`         | `post-meta/`            | dynamic | Post meta display                                    |
| `taxonomy`           | `taxonomy`              | `Taxonomy`         | `taxonomy/`             | dynamic | Taxonomy term display                                |
| `breadcrumbs`        | `breadcrumbs`           | `Breadcrumbs`      | `breadcrumbs/`          | dynamic | Breadcrumb navigation                                |
| `table_of_contents`  | `table-of-contents`     | `TableOfContents`  | `table-of-contents/`    | dynamic | Auto-generated ToC from headings                     |

### Layout / containers

| Registry key         | WP name                 | PHP class          | JS dir                  | Type    | Notes                                              |
|----------------------|-------------------------|--------------------|-------------------------|---------|----------------------------------------------------|
| `row`                | `row`                   | `Row`              | `row/`                  | static  | Row + column container system                       |
| `column`             | `column`                | `Column`           | `column/`               | static  | Child of row                                         |
| `flex_container`     | `flex-container`        | `FlexContainer`    | `flex-container/`       | static  | Newer flexbox-based container                        |
| `advanced_navigation`| `advanced-navigation`   | `AdvancedNavigation` | `advanced-navigation/` | static  | Mega menu-style navigation                           |
| `advanced_tabs`      | `advanced-tabs`         | `AdvancedTabs`     | `advanced-tabs/`        | static  | Tabs container with child `tab`                     |

### Advanced media

| Registry key         | WP name                 | PHP class          | JS dir                  | Type    | Notes                                              |
|----------------------|-------------------------|--------------------|-------------------------|---------|----------------------------------------------------|
| `advanced_image`     | `advanced-image`        | `AdvancedImage`    | `advanced-image/`       | static  | Image with hotspots, captions, advanced styling      |
| `advanced_video`     | `advanced-video`        | `AdvancedVideo`    | `advanced-video/`       | static  | Video with custom controls                           |
| `icon`               | `icon`                  | `Icon`             | `icon/`                 | static  | Single icon block                                    |
| `google_map`         | `google-map`            | `GoogleMap`        | `google-map/`           | dynamic | Google Maps embed                                    |
| `call_to_action`     | `call-to-action`        | `CallToAction`     | `call-to-action/`       | static  | Marketing CTA section                                |
| `text`               | `text`                  | `Text`             | `text/`                 | static  | Rich text block                                      |

### Forms

| Registry key                | WP name                   | PHP class          | JS dir                      | Type    | Notes                                              |
|-----------------------------|---------------------------|--------------------|-----------------------------|---------|----------------------------------------------------|
| `form`                      | `form`                    | `Form`             | `form/`                     | dynamic | Most-touched block (218 commits). Integration with plugins |
| `form_text_field`           | `form-text-field`         | `FormTextField`    | `form-text-field/`          | dynamic | Child of `form`                                     |
| `form_textarea_field`       | `form-textarea-field`     | —                  | `form-textarea-field/`      | dynamic | Child of `form`                                     |
| `form_email_field`          | `form-email-field`        | —                  | `form-email-field/`         | dynamic | Child of `form`                                     |
| `form_number_field`         | `form-number-field`       | —                  | `form-number-field/`        | dynamic | Child of `form`                                     |
| `form_select_field`         | `form-select-field`       | —                  | `form-select-field/`        | dynamic | Child of `form`                                     |
| `form_checkbox_field`       | `form-checkbox-field`     | —                  | `form-checkbox-field/`      | dynamic | Child of `form`                                     |
| `form_radio_field`          | `form-radio-field`        | —                  | `form-radio-field/`         | dynamic | Child of `form`                                     |
| `form_datetime_picker`      | `form-datetime-picker`    | —                  | (pro / future)              | dynamic | Child of `form` (registered in manifest)            |
| `form_country_field`        | `form-country-field`      | —                  | (pro / future)              | dynamic | Child of `form`                                     |
| `form_phone_field`          | `form-phone-field`        | —                  | (pro / future)              | dynamic | Child of `form`                                     |
| `form_recaptcha`            | `form-recaptcha`          | —                  | (pro / future)              | dynamic | Child of `form`                                     |
| `wpforms`                   | `wpforms`                 | `WPForms`          | `wpforms/`                  | dynamic | WPForms plugin wrapper                               |
| `fluent_forms`              | `fluent-forms`            | `FluentForms`      | (integration)               | dynamic | Fluent Forms integration                             |

Form block logic (PHP) lives in `includes/Integrations/Form.php` — not in
`includes/Blocks/Form.php` alone. That's where submission, validation, and
email go.

### WooCommerce

| Registry key         | WP name                 | PHP class          | JS dir                  | Type    | Notes                                              |
|----------------------|-------------------------|--------------------|-------------------------|---------|----------------------------------------------------|
| `woo_product_grid`   | `woo-product-grid`      | `WooProductGrid`   | `woo-product-grid/`     | dynamic | Product grid query                                  |
| `add_to_cart`        | `add-to-cart`           | `AddToCart`        | `add-to-cart/`          | dynamic | Single product add-to-cart button                   |
| `product_details`    | `product-details`       | `ProductDetails`   | `product-details/`      | dynamic | Product description/meta                            |
| `product_images`     | `product-images`        | `ProductImages`    | `product-images/`       | dynamic | Product image gallery                               |
| `product_price`      | `product-price`         | `ProductPrice`     | `product-price/`        | dynamic | Product price display                               |
| `product_rating`     | `product-rating`        | `ProductRating`    | `product-rating/`       | dynamic | Product star rating                                 |
| `price`              | `price`                 | `price.php`        | `price/`                | dynamic | Price utility block                                 |

## Registered-in-manifest but late-addition / pro-facing

These appear in `includes/blocks.php` but may be pro-only or newer:

- `advanced_search`
- `data_table`
- `timeline_slider`
- `news_ticker`
- `woo_product_carousel`
- `multicolumn_pricing_table`
- `fancy_chart`
- `stacked_cards`
- `testimonial_slider`
- `off_canvas`
- `loop_builder`
- `post_template`
- `loop_pagination`
- `mega_menu`
- `business_hours`
- `animated_wrapper`

If a block name appears in `includes/blocks.php` but has no matching
`src/blocks/<kebab>/` directory, it's registered by the pro plugin and
included in the manifest for visibility/UI purposes.

## Ranking by commit frequency (free plugin only, from commit-message mentions)

Highest-touched blocks, useful for "what's volatile":

1. form — 218 commits
2. button — 217
3. slider — 180
4. accordion — 170
5. post-grid — 138
6. infobox — 106
7. image-gallery — 93
8. team-member — 84
9. toggle-content — 76
10. social — 71
11. flipbox — 64
12. post-carousel / countdown — 61
13. testimonial — 58
14. advanced-heading — 55
15. pricing-table — 50

Top JS file hotspots (individual files, not block totals):

- `src/blocks/image-gallery/src/edit.js` — 61 commits
- `src/blocks/image-gallery/src/inspector.js` — 57
- `src/blocks/timeline/src/inspector.js` — 43
- `src/blocks/timeline/src/style.scss` — 41
- `src/blocks/lottie-animation/src/edit.js` — 38
- `src/blocks/slider/src/inspector.js` — 34
- `src/blocks/form/src/edit.js` — 33
- `src/blocks/lottie-animation/src/frontend.js` — 32
- `src/blocks/accordion/src/edit.js` — 31

If you're touching any of these, history has opinions about the right shape.
`git log --oneline -- <file>` before refactoring.

## How to find a block quickly

```bash
EB="<free path>"
# By slug:
ls "$EB/src/blocks/" | grep -i "slider"
ls "$EB/includes/Blocks/" | grep -i "Slider"

# By PHP class name:
grep -rn "class Slider " "$EB/includes/Blocks/"

# By WP block name:
grep -rn "essential-blocks/slider" "$EB/src/blocks/slider/"
```

And for the template used by a dynamic block render:

```bash
grep -rn "require.*views" "$EB/includes/Blocks/Slider.php"
```

## Block category taxonomy

From `includes/blocks.php`:

- `content` — accordion, button, call_to_action, flipbox, infobox, advanced_heading, text, …
- `creative` — countdown, image_comparison, lottie, shape_divider, slider, …
- `marketing` — call_to_action, pricing_table, cta variants
- `query` — post_grid, post_carousel, taxonomy, …
- `forms` — form + form_* field children
- `woocommerce` — woo_product_grid, product_details, etc.

These drive the block inserter category in the editor.
