+++
title = "PyData NYC 2018: Developing Dashboard Applications using Bokeh"
date = 2018-10-17
type = "slides"
tags = ["presentation"]
summary = "A presentation about using Bokeh to create streaming dashboards at PyData NYC 2018, featuring an example Bokeh application that performs face detection using a webcam and OpenCV."
+++
<section data-markdown>
	<textarea data-template>
		## Developing Dashboard Applications with Bokeh

		<img src="/img/pydata_nyc_2018/bokeh_logo.svg" width="100" height="100" style="background:none; border:none; box-shadow:none;"/>

		*Luke Canavan*
	</textarea>
</section>
<section>
	<section data-markdown>
		<textarea data-template>
			Bokeh seeks to connect PyData tools and users to web-based, interactive visualization

			<img src="/img/pydata_nyc_2018/kepler.gif">
			[KeplerGO/lightkurve](https://github.com/KeplerGO/lightkurve)
		</textarea>
	</section>
	<section data-markdown>
		<textarea data-template>
			### Bokeh 1.0 Announcement

			Features include:
			* Highly customizable plotting primitives
			* Interactive plot tools (pan, hover, zoom, etc)
			* HTML Widgets (sliders, dropdowns, etc)
			* Native support for network graphs and geo data
			* Jupyter Notebook, Jupyter Lab, Zeppelin support
			* JS and Python callbacks on data changes and events
			* Bokeh Server
		</textarea>
	</section>
</section>
<section>
	<section data-markdown>
		<textarea data-template>
			### Outline

			* Building visualizations from primitives
			* Layouts with bokeh.layouts and custom templates
			* Styling using Palettes and Themes
			* Running as a server application
		</textarea>
	</section>
	<section data-markdown>
		<textarea data-template>
			### Goal

			<img src="/img/pydata_nyc_2018/screenshot.png" height=481 width=830/>
		</textarea>
	</section>
</section>
<section>
	<section data-markdown>
		<textarea data-template>
			### Building visualizations from primitives
		</textarea>
	</section>
	<section data-markdown>
		<textarea data-template>
			Pick what graphical primitives to use, provide the data, and specify how to
			map visual properties to data fields. Bokeh will take care of the rest.
		</textarea>
	</section>
	<section data-markdown>
		<textarea data-template>
			### Bokeh / BokehJS Architecture
			<img src="/img/pydata_nyc_2018/bokeh_models.svg" height=200 width=1000/>
		</textarea>
	</section>
	<section data-markdown>
		<textarea data-template>
			### Using the bokeh.models API

			```
			$ from bokeh.models import Circle

			$ circle = Circle(x=5, y=10, fill_color="red")

			$ circle.to_json(include_defaults=False)

			> {
				"angle": {
					"units": "rad",
					"value": 0
				},
				"fill_alpha": {
					"value": 1
				},
				"fill_color": {
					"value": "red"
				},
				"id": "551a8ecc-1059-4478-9ddd-bac1e69f6c76",
				"js_event_callbacks": {},
				"js_property_callbacks": {},
				"line_alpha": {
					"value": 1
				},
				"line_cap": "butt",
				"line_color": {
					"value": "black"
				},
				"line_dash": [],
				"line_dash_offset": 0,
				"line_join": "bevel",
				"line_width": {
					"value": 1
				},
				"name": null,
				"radius": null,
				"radius_dimension": "x",
				"size": {
					"units": "screen",
					"value": 4
				},
				"subscribed_events": [],
				"tags": [],
				"x": {
					"value": 5
				},
				"y": {
					"value": 7
				}
			}
			```
		</textarea>
	</section>
	<section data-markdown>
		<textarea data-template>
			### Write data driven visualizations

			```
			$ from bokeh.models import ColumnDataSource

			$ data = {
				"x": [...],
				"y": [...],
				"color": [...]}

			### From Dict[str, list]

			$ source = ColumnDataSource(data)

			### From Pandas DataFrame

			$ df = pd.DataFrame(data)
			$ source = ColumnDataSource(df)
			```
		</textarea>
	</section>
	<section data-markdown>
		<textarea data-template>
			### Map visual properties to data fields

			```
			$ source = ColumnDataSource({
				"x": [...],
				"y": [...],
				"color": [...]}

			$ plot.add_glyph(
				source,
				Circle(x='x', y='y', fill_color='color'))
			```
		</textarea>
	</section>
	<section data-markdown>
		<textarea data-template>
			To Reiterate:

			Pick what graphical primitives to use, provide the data, and specify how to
			map visual properties to data fields. Bokeh will take care of the rest.

			More reading at [Enjoying the bokeh.models API](https://bokeh.github.io/blog/2017/7/5/idiomatic_bokeh/)
		</textarea>
	</section>
	<section data-markdown>
		<textarea data-template>
			### Face Detection Example
		</textarea>
	</section>
</section>
<section>
	<section data-markdown>
		<textarea data-template>
			### Layouts with bokeh.layouts and custom templates
		</textarea>
	</section>
	<section data-markdown>
		<textarea data-template>
			Plots and Widgets can be responsive

			<img src="/img/pydata_nyc_2018/responsive.gif" width="461", height="444">
		</textarea>
	</section>
	<section data-markdown>
		<textarea data-template>
			The bokeh.layouts API offers a "rows and columns"-based layout interface

			```
			from bokeh.layouts import layouts

			l = layout([
				[bollinger],
				[sliders, plot],
				[p1, p2, p3],
				], sizing_mode='stretch_both')
			```
		</textarea>
	</section>
	<section data-markdown>
		<textarea data-template>
			### Using bokeh.layouts

			<img src="/img/pydata_nyc_2018/layout.png" width="668", height="530"/>
		</textarea>
	</section>
	<section data-markdown>
		<textarea data-template>
			### Using custom templates

			* Support for embedding roots into custom Jinja templates
			* New in Bokeh 0.13.0, you can individually lay out Document items...
		</textarea>
	</section>
	<section data-markdown>
		<textarea data-template>
			Bokeh + CSS Layout Models = Amazing Layouts

			<img src="/img/pydata_nyc_2018/cssgrid.png" style="background:white"/>

			[CSS Grid](https://css-tricks.com/snippets/css/complete-guide-grid/)
		</textarea>
	</section>
	<section data-markdown>
		<textarea data-template>
			Dask Distributed's diagnostic UI uses CSS Grid

			<img src="/img/pydata_nyc_2018/dask_full.png"/>
		</textarea>
	</section>
	<section data-markdown>
		<textarea data-template>
			Dask Distributed's diagnostic UI uses CSS Grid

			<img src="/img/pydata_nyc_2018/dask_half.png" width="462" , height="466" />
		</textarea>
	</section>
	<section data-markdown>
		<textarea data-template>
			### Face Detection Example
		</textarea>
	</section>
</section>
<section>
	<section data-markdown>
		<textarea data-template>
			### Styling using Palettes and Themes
		</textarea>
	</section>
	<section data-markdown>
		<textarea data-template>
			Bokeh offers many aesthetically pleasing color palettes including the Brewer
			and D3 palettes.

			<img src="/img/pydata_nyc_2018/d3_palette.png" width="692" height="400" />

			[Made with Holoviews](http://holoviews.org/gallery/demos/bokeh/autompg_violins.html#bokeh-gallery-autompg-violins)
		</textarea>
	</section>
	<section data-markdown>
		<textarea data-template>
			Bokeh also offers perceptually uniform color palettes to allow users to map data
			to color ranges.

			<img src="/img/pydata_nyc_2018/fire_palette.png" width="500" height="400" />

			[Made with Datashader](http://datashader.org/getting_started/1_Introduction.html)
		</textarea>
	</section>
	<section data-markdown>
		<textarea data-template>
			Themes are great for maintaining consistent style across several plots

			```
			attrs:
				Axis:
					axis_line_color: "#49483E"
					axis_label_text_color: "#888888"
					major_label_text_color: "#888888"
					major_tick_line_color: "#49483E"
					minor_tick_line_color: "#49483E"
				Legend:
					border_line_color: "#49483E"
					background_fill_color: "#282828"
					label_text_color: "#888888"
				Plot:
					background_fill_color: "#282828"
					border_fill_color: "#282828"
					outline_line_color: "#49483E"
			```
		</textarea>
	</section>
	<section data-markdown>
		<textarea data-template>
			Community-created Themes
			<table>
				<tr>
					<th>
						<p>Caliber</p>
						<img src="/img/pydata_nyc_2018/caliber.png" width="325" height="187"/>
					</th>
					<th>
						<p>Monokai Dark</p>
						<img src="/img/pydata_nyc_2018/monokai.png" width="325" height="187"/>
					</th>
					<th>
						<p>Minimal Dark</p>
						<img src="/img/pydata_nyc_2018/dark.png" width="325" height="187"/>
					</th>
				</tr>
			</table>
		</textarea>
	</section>
</section>
<section>
	<section data-markdown>
		<textarea data-template>
			### Running as a Bokeh server application
		</textarea>
	</section>
	<section data-markdown>
		<textarea data-template>
			Bokeh Server keeps Python and JS models in sync.

			<img src="/img/pydata_nyc_2018/bokeh_server.svg" height=300 width=1000/>
		</textarea>
	</section>
	<section data-markdown>
		<textarea data-template>
			Python callbacks can executed on property changes

			```
			...

			def update(attr, old, new):
				plot.title.text = new

			x = Select(title='X-Axis', value='mpg', options=columns)
			x.on_change('value', update)
			```
		</textarea>
	</section>
	<section data-markdown>
		<textarea data-template>
			Document-level Python callbacks are intended for server-driven updates
			like pushes and polling

			```
			def stream_image():
				...

				source.data["image"] = [image]

			doc.add_periodic_callback(acquire_image, 100)
			```
		</textarea>
	</section>
</section>
