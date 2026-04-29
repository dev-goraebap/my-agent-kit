---
source_url: https://m3.material.io/styles/icons/applying-icons
fetched_at: 2026-04-29
section: styles
---

Icons
Icons are small symbols to easily identify actions and categories
Overview
Designing icons
Applying icons
On this page
Icons
Icon & Material Symbol styles
Customizing Symbols
Using Material Symbols with typography
Accessibility
Localizing icons

Icon & Material Symbol styles

Material Symbols are the new default, and are available in three styles: outlined, rounded, and sharp. (The legacy Material Icons continue to be available, but don’t have the variable font capabilities of Material Symbols.)

Outlined style 

Outlined symbols use stroke and fill attributes for a light, clean style that works well in dense UIs. The stroke weight of outlined icons can be adjusted to complement or contrast the weight of your typography.

Outlined style

2dp outlined icons remain readable across sizes and applications

For optimal legibility and recognition, some symbols should remain filled, such as full body human icons or proprietary icons

The lighter stroke weight of these outlined symbols mirrors the thin lines of the app’s typography

Rounded and sharp styles

Rounded symbols use a corner radius that pairs well with brands that use heavier typography, curved logos, or circular elements to express their style.

Sharp symbols display corners with straight edges, for a crisp style that remains legible even at smaller scales. These rectangular shapes can support brand styles that aren’t well-reflected by rounded shapes.

Rounded-style icons

Sharp-style icons

Corner radii for round icons

Square corner radii for sharp icons

This app uses rounded buttons and round icons

The 0dp corner radius of the sharp icon set echoes this app’s rectangular design details

Customizing Symbols

Material Symbols have four adjustable stylistic variable font attributes called axes. An axis is a typographic term referring to the attribute of a symbol that can be altered to create visual variations. 

Each style symbol contains four axes: weight, fill, grade, and optical size.

Weight

Weight defines the symbol’s stroke weight, with a range of weights between thin (100) and bold (700). Weight can also affect the overall size of the symbol. 

A symbol in a range of weights

400 regular-weight symbols

close
Don’t

Don't use the lightest weight for standard-size (24dp) icons. The minimum weight for this size should be 200.

exclamation
Caution

Be careful using excessive weight for standard 24dp symbols

check
Do

Apply weights consistently

close
Don’t

Don’t mix different weights

Fill

Fill gives you the ability to transition from a more outlined style to a reversed or more filled style.

A fill attribute can be used to convey a state of transition, such as unfilled and filled states. Values range from 0 to 1, with 1 being completely filled. Along with weight, fill is a primary attribute that impacts the overall look of a symbol.

Unfilled symbols with fill set to 0

Filled symbols with fill set to 1

Bottom navigation with filled symbols in selected and unselected states

Grade

Weight and grade affect a symbol’s thickness. Adjustments to grade are more granular than adjustments to weight and have a smaller impact on the size of the symbol.

Grade is also available in some text fonts. Grade levels between text and symbols can be matched for a harmonious visual effect. For example, if the text font has a -25 grade value, the symbols can match it with a suitable value of -25. 

At grade 0, the thickness of the symbol does not change
At negative grade, the thickness of the symbol appears lighter

Grade can also compensate for visual bleed, which is when images can look bigger or smaller depending on the color contrast. To match the apparent icon size, the default grade for a dark icon on a light background is 0, and -25 for a light icon on a dark background.

Icon button featuring a 0 default grade symbol in light UI

Icon button featuring a negative grade symbol in dark UI

To make strokes heavier and more emphasized, use positive value grade, such as when representing an active icon state.

An icon with active state using positive value grade for emphasis

Optical sizes

Optical sizes range from 20dp to 48dp.

For the image to look the same at different sizes, the stroke weight (thickness) changes as the icon size scales. Optical size offers a way to automatically adjust the stroke weight when you increase or decrease the symbol size.

Four optical sizes, 20dp, 24dp, 40dp, 48dp

Traditionally, icons are resized from a 24dp source vector, resulting in a large scaled icon that’s too heavy compared to the original. With the optical size axis, you can maintain the stroke weight (thickness) as the icon size grows.

Material icon

Material Symbol

Use 20dp optical size value for dense layouts on desktop

Use larger size 40dp–48dp symbols when primary actions need to be highlighted

Using Material Symbols with typography

Material Symbols are designed with similar considerations to typefaces, and often appear alongside text. Choosing the right icon set can tie the content of an interface together, enhancing the cohesive branded feel of your product.

Match the optical weight and size of text and icon to ensure consistency

check
Do

Use the same size for your Material Symbols and text

close
Don’t

Don’t mix the sizes of your symbol and text

check
Do

Use the same optical weight for your symbol and text

close
Don’t

Don’t use different optical weights for Material Symbols and text

check
Do

Shift down the baseline of symbols to approximately 11.5% of the text size

close
Don’t

Don’t use the same baseline for Material Symbols and text

Accessibility

Learn more about making your icons more accessible.

Icons with a label text

Label text provides short, meaningful descriptions when symbols are more abstract. This can prove helpful in the case of navigation.

Label text provides short descriptions, especially useful for navigation

Use caution if icons are displayed without labels. Icon meaning should always be unambiguous and accessible for all users. Text labels can be omitted in specific circumstances where reduced visual impact is necessary.

Small icons

Material Symbols can scale up or down in size without a loss of fidelity. Simple symbols, like stars for ratings, can be used on their own at any size, as long as they remain identifiable.

Other symbols should have an accompanying text label below 20dp to ensure their meaning is clear and to maintain accessibility. These symbols include:

Complex icons, which are highly detailed or have multiple parts

Icons with a key action, which are essential to using the product

Target size

Adequate space should surround icons to allow legibility and interaction.

Symbols of 24dp should have a target size of 48dp by default.

Measurements
Placement

When a mouse and keyboard are the primary input methods, measurements may be condensed to accommodate denser layouts.

A 20dp size symbol can use a target size of 40dp.

Measurements
Placement

Localizing icons

To make sure iconography translates effectively in local markets, test it across age groups, cultures, and languages, and follow these best practices:

Use labels when icons and symbols are more abstract
Remember that navigation items must have labels for clarity and accessibility
Consider tech knowledge: people who use the internet a lot may have different understandings of icons than people who use the internet less

Translate icons for local markets. For example, different locales may prefer a cart, bag, or basket for checkout experiences.

Cultural influence of colors and symbols

Color carries cultural significance and can convey different emotions in different cultures. White is commonly associated with purity in western cultures but symbolizes mourning in some eastern cultures.

Consider cultural interpretations of symbols. In many western cultures, owls represent wisdom, while some eastern cultures view them as a negative omen. When using or creating symbols, be mindful that their meanings can vary significantly across cultures.

Think about how color translates. Some locales use red as a warning color, while others use green.
