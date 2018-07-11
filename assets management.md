# Assets management

## Where to put assets

Images are stores in an `assets` folder, inside the `src` folder.

Images are imported via an `require` statement at the top of each module, from the assets folder.

## How to preload images managed by Webpack (i.e. included in my src/ directory)

Problem: If for example I am showing some images only conditionally, I want the loading time to be minimal or non-existent.
There are techniques for preloading images, but a lot of them don't work if you manage assets as part of your Webpack system (which is recommended as assets are hashed, and you get build-time errors if any are missing).

Solution: 
- Create an `AssetsLoader` component, inside the `./assets/` folder.
- The job of this component is to:

	- Import _all_ image assets used throughout the app, and export them immediately
	- Render _all_ image assets within a div that is `visibility: hidden`, 0 `height`, 0 `width` and `overflow: hidden` so that the images do not show there – but they are pre-loaded
	
- An `index.ts` file in the `assets/` folder forwards all the named exports from `AssetsLoader`: the component itself, but also all the image assets which it imports.

- All other components throughout the app which need image assets, import the constants that hold the right image asset from `./assets/`.

- `App.tsx` imports the `AssetsLoader` component directly (it's the only component that does), and renders it right at the bottom of the page, so that all images are pre-loaded as part of the initial app loading.


## How I name assets

All image name begin with `PROTOIMG_`. This allows me to easily find all the slices for the prototying inside the source sketch file by searching the document for `PROTOIMG_`.

The next bit of the file name says what the type of the image is, so that all images are organised in types. Eg `nav`, `table`, `graph`, etc.

The next gets get increasingly more specific about what this image represents, and how it's different from others.

If I export different versions of the same image (eg to represent different states of the app), only the very last bit of the filename should change.

Eg:

```
PROTOIMG_graph_customers
PROTOIMG_graph_salesValue
PROTOIMG_graph_spendPerCustomer
PROTOIMG_graph_spendPerVisit

PROTOIMG_kpiTree

PROTOIMG_nav_footer
PROTOIMG_nav_header
PROTOIMG_nav_tabs

PROTOIMG_table_customerTypes_salesValue
PROTOIMG_table_regions_salesValue
PROTOIMG_table_storeFormats_salesValue

PROTOIMG_table_subcategories_customers
PROTOIMG_table_subcategories_salesValue
PROTOIMG_table_subcategories_spendPerCustomer
PROTOIMG_table_subcategories_spendPerVisit
```

The variables holding each assets are always named exactly like the asset, except for the .png file extension.

## Where prototype images come from

All assets come from __1 sketch file__ named `PROTOMASTER.sketch`. 
I.e. if I want to add something new, I either produce it in here, or I paste it in that sketch file and export the slices from that sketch file.

That sketch file is _inside_ the git repository, in the `sketchsource` folder.

## Where the asset names come from

If an asset comes from Sketch, it is __named within sketch__ (i.e. the name is the name of the slice I use to export the asset) and __this asset name is never modified later__.

This allows me to always easily be able to find the source sketch elements of an image, quickly do modifications, reexport the asset in the same location, and see the changes immediately in the app without any extra work. 

## How I orgamyse my sketch file

When I need to export variations of a design (eg to represent different states of the same component), I put each different variation on a different sketch artboard to the right on the first one. This allows me to easily see variations.

![](./assets/organisingAltVersionsInSketch.png)
![](./assets/organisingAltVersionsInSketch2.png)



## How to show different images based on some condition

1. Import all the image assets the component might display and assign them to different constants:

```
import { PROTOIMG_graph_salesValue, PROTOIMG_graph_customers, PROTOIMG_graph_spendPerCustomer, PROTOIMG_graph_spendPerVisit, PROTOIMG_table_subcategories_customers, PROTOIMG_table_subcategories_salesValue, PROTOIMG_table_subcategories_spendPerCustomer, PROTOIMG_table_subcategories_spendPerVisit, PROTOIMG_table_customerTypes_salesValue, PROTOIMG_table_regions_salesValue, PROTOIMG_table_storeFormats_salesValue, PROTOIMG_kpiTree } from './../../../assets/'
```

2. Use an IIFE with a switch block returning the right constant, right inline when specifying the `src` of the `<img />` html element.

To write an IIFE, write the whole function definition in between `(...)`, then add calling parentheses at the end.

```
<img 
	src={
		switch (appState.selectedMeasure) {
			case 'Sales value':
			case 'Basket penetration':
				return PROTOIMG_graph_salesValue
			case 'Spend per customer':
			case 'Frequency of purchase':
			case 'Sales units':
				return PROTOIMG_graph_spendPerCustomer
			...
			default:
				const _exhaustiveCheck: never = appState.selectedMeasure
		}
	} 
/>
```

Note: Remember that switch cases fall through from one another by default in JS.

Note: I tried to replace this IIFE with an assetGetter function, but I didn't manage as it wasn't clear what the return type should be (I don't think there's a clear type in TS for an image asset held in a variable), and returning an `<img src={} />` html element didn't work.



# Using Sketch symbols to rapidly iterate prototype assets

## Create symbols, then make different instances of the symbol with different data / content

Don’t just duplicate artboards to create different state-versions of data/content for an app.
Instead, take a few minutes to create a symbol, which I will instantiate several times with different data.

Put these symbols in the ’Symbols’ page of the ‘PROTO_MASTER’ doc.

![](./assets/assetsManagement-symbolsAndWhereTheyLive.png)

## Creating symbols so that they're easy to override

#### 1 Organise and order:
- Organise the elements into clear groups
- Within each group, order the elements, following a natural reading order (left to right, then top to bottom)

#### 2 Name:
- Name the elements with short, consistent, easily scannable names – so I can find and override these properties easier
- Mark any text element or nested symbol that I will not need to override for the different instances as private (_elementName)

![](./assets/assetsManagement-organisationAndNaming.png)

#### 3 Text elements overriding: 
- Make sure that text fields can accomodate long enough content for the different instances’ data. (eg I can use align right, or allign bottom, with ample boxes)

![](./assets/assetsManagement-spaceAndAlignmentForTextContentOverriding.png)

#### 4 Nested symbols overriding: 
If instances customisation will require the use of a different element, create a separate atomic symbol (along with alternative versions). With just that element, and nest it in the symbol. If a symbol has nested symbols, I can override each of the nested symbols on the symbol’s instance.

![](./assets/assetsManagement-nestedSymbolsForOverriding.png)

#### 5 Use nested symbols overriding because text styles overriding isn’t yet possible in Sketch
Sketch allows symbol instances to have a different text content, but not a different text style.
The workaround is to create a symbol for the text element (and alternative versions of the text symbol with different styles), and then to nest that symbol in my symbol.
To do this, in the ‘Symbols’ page of the ‘PROTO_MASTER’, create an artboard that has the same size as the text element I want to symbolise. Then copy and paste the text element in it, and convert the artboard into a symbol. Then back into the higher-order symbol, replace the element with the symbol (i.e. nesting a symbol in place of an element).

![](./assets/assetsManagement-alternativeSymbolsForOverridableTextStyles.png)

![](./assets/assetsManagement-overridingSymbolsAsTextStyles.png)

Note: I can create these more atomic symbols for nesting either in the ‘Symbols’ page of the ‘PROTO_MASTER’ doc, or in the project’s shared library (if I think that the symbol will be reused somewhere else).

### Case 1: When text content and nested symbol overriding _are_ sufficient to customise the symbols with different data/content

Then in the ‘PROTOSVG_ exports’ page
    - create an artboard that’s the same size as the symbol
    - insert the symbol on the artboard
    - use text overrides, and nested symbol overrides, to customise each instance with the right data/content

### Case 2: When text content and nested symbol overriding _aren't_ sufficient to customise the symbols with different data/content

The customisation of the instances may require things that are not easy with text or nested symbols override:
eg
- different line graphs
- different size bars

When that’s the case:
In the ‘Symbols’ page of ‘PROTO_MASTER’:
1. Create/Paste a model for the future symbol as an artboard
2. Make it all perfect
3. Make a backup
4. Remove the data / anything that I wouldn’t be able to customise via text/nested symbols overriding
5. Turn this artboard without data into a symbol, eg `trendGraph/noData`
6. Still in ‘Symbols’ page of ‘PROTO_MASTER’, just under the symbol, create an artboard with the same size, and inser the symbol in it. Call this `trendGraph/populatedTemplate`. (Note: this is just an artboard with a symbol in it, not a symbol)
7. Populate the `trendGraph/populatedTemplate` artboard with data/content, on top of the inserted `trendGraph/noData` symbol

Then, in the ‘PROTOSVG_ exports’ page:
8. Copy and paste the `trendGraph/populatedTemplate` into the ‘PROTOSVG_ exports’ page (Nb: I’m not inserting the `trendGraph/noData` symbol, but coping and pasting the `trendGraph/populatedTemplate` artboard which has the `trendGraph/noData` symbol in it).
9. Customise the populated data for each instance

![](./assets/assetsManagement-symbolsWithNoDataAndAccompanyingPopulatedTemplateArtboardsForCopyingAndPasting)

![](./assets/assetsManagement-populatedTemplateImportedAndCustomised.png)



