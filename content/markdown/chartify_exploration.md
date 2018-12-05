+++
title = "Chartify: A quick review"
date = 2018-12-02
type = "blog"
tags = ["chartify"]
+++

<link
    href="https://cdn.pydata.org/bokeh/release/bokeh-1.0.1.min.css"
    rel="stylesheet" type="text/css">

<script src="https://cdn.pydata.org/bokeh/release/bokeh-1.0.2.min.js"></script>

[Chartify](https://github.com/spotify/chartify) is a new plotting library that was recently open-sourced by Spotify Labs. You can read their announcement article [here](https://labs.spotify.com/2018/11/15/introducing-chartify-easier-chart-creation-in-python-for-data-scientists/). Chartify is intended to make it easy for Python users to create standard chart types, including line, bar and area charts, and is built on top of [Bokeh](https://bokeh.pydata.org/en/latest/). As a Bokeh core contributor, I quickly experimented with Chartify to see what it's like.

tl;dr I'm impressed. Chartify offers a clean API to ingest tidy data and generate a variety of visually pleasing charts, while also exposing the underlying Bokeh figure for further customization. I'm excited about this addition to the Python data visualization ecosystem.

![Horizontal Histogram](/img/chartify_review/chartify_histogram.png)
(taken from https://github.com/spotify/chartify/blob/master/examples/Examples.ipynb)

### Why does Chartify build on Bokeh?

Bokeh is a tool for creating web-based, interactive visualizations and offers a lot of primitives (like lines and circles) that users combine into highly customized visualizations. However, using primitives means that users may be required to do extra data manipulation to create their desired plot. For example, there's no ``from bokeh import StackedBarChart``. Users can certainly create such a chart using Bokeh, but doing so requires figuring out how to transform their data into beginning and end positions for the stacked bars. Chartify aims to abstract away this data transformation step for users making standard chart types.

### Using Tidy Data:

Chartify consumes tidy data, a data formatting concept that originated in the R ecosystem. You can read the whole explanation [here](https://cran.r-project.org/web/packages/tidyr/vignettes/tidy-data.html), but synopsis is that a tidy dataset is one that is structured where:

* Each variable forms a column
* Each observation forms a row
* Each type of observational unit forms a table

To fully understand, it might be easier to look at an example of each:

*Tidy Data Example*:

<table>
  <thead>
    <tr>
      <th></th>
      <th>date</th>
      <th>country</th>
      <th>fruit</th>
      <th>unit_price</th>
      <th>quantity</th>
      <th>total_price</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2017-10-21</td>
      <td>US</td>
      <td>Banana</td>
      <td>0.303711</td>
      <td>4</td>
      <td>1.214846</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2017-05-30</td>
      <td>JP</td>
      <td>Banana</td>
      <td>0.254109</td>
      <td>4</td>
      <td>1.016436</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2017-05-21</td>
      <td>CA</td>
      <td>Banana</td>
      <td>0.268635</td>
      <td>4</td>
      <td>1.074539</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2017-09-18</td>
      <td>BR</td>
      <td>Grape</td>
      <td>2.215277</td>
      <td>2</td>
      <td>4.430554</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2017-12-08</td>
      <td>US</td>
      <td>Banana</td>
      <td>0.308337</td>
      <td>5</td>
      <td>1.541687</td>
    </tr>
  </tbody>
</table>

*Untidy Data Example*:

<table>
  <thead>
    <tr>
      <th>country</th>
      <th>BR</th>
      <th>CA</th>
      <th>GB</th>
      <th>JP</th>
      <th>US</th>
    </tr>
    <tr>
      <th>fruit</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Apple</th>
      <td>57</td>
      <td>144</td>
      <td>177</td>
      <td>65</td>
      <td>165</td>
    </tr>
    <tr>
      <th>Banana</th>
      <td>30</td>
      <td>222</td>
      <td>113</td>
      <td>232</td>
      <td>479</td>
    </tr>
    <tr>
      <th>Grape</th>
      <td>54</td>
      <td>86</td>
      <td>59</td>
      <td>52</td>
      <td>81</td>
    </tr>
    <tr>
      <th>Orange</th>
      <td>74</td>
      <td>207</td>
      <td>97</td>
      <td>75</td>
      <td>409</td>
    </tr>
  </tbody>
</table>

You can see that each row in the tidy dataset contains a unique observation, composed of values for each variable. In the untidy dataset, each row corresponds to the summary of a different type of fruit and not
unique observations.

Data analysis tools like Pandas are generally designed to consume data that matches this standard. Since Chartify is a Python library, you can read about about tidying data in Pandas from Pandas core contributor Tom Augspurger [here](https://tomaugspurger.github.io/modern-5-tidy.html). This is especially relevant because Chartify ingests tidy Pandas DataFrames for plotting, which is hugely valuable because users don't have to do any special data transformation in order create visualizations.

### The Chartify API

Chartify users create a ``chartify.Chart`` object and specify one of a few enumerated axis types for the x and y axes. The resulting ``Chart`` object will contain a set of appropriate plotting methods for your axis pair type. For example, using a ``"datetime"`` x-axis and ``linear`` y-axis means that a line chart is a good idea and bar chart is not, because bar charts are typically intended for categorical data. I think this is great - Bokeh tries very hard to help users make effective visualizations by having nice defaults and I think these opinionated guardrails are good.

*The allowed axis types*:

x_axis_type (enum, str):

- 'linear'
- 'log'
- 'datetime'
- 'categorical'
- 'density'

y_axis_type (enum, str):

- 'linear'
- 'log'
- 'categorical'
- 'density'

As of release 2.3.5, Chartify offers the following chart types for the corresponding x and y axis types:

<table>
  <tr>
    <th>X Axis Below/Y Axis Right</th>
    <th>linear/log/datetime</th>
    <th>categorical</th>
    <th>density</th>
  </tr>
  <tr>
    <th>linear/log/datetime</th>
    <td>line, scatter, text, area</td>
    <td>bar, lollipop, parallel</td>
    <td>kde, histogram</td>
  </tr>
  <tr>
    <th>categorical</th>
    <td>bar, lollipop, parallel</td>
    <td>heatmap</td>
    <td>kde, histogram</td>
  </tr>
  <tr>
    <th>density</th>
    <td>kde, histogram</td>
    <td>kde, histogram</td>
    <td>hexbin</td>
  </tr>
</table>

(Note: both ``area`` and ``bar`` include stacked area and bar charts)

While there's endless the potential to add more, I think Chartify more than covers the necessary charts for general report generation.

### Using the "chartify.Chart.plot" methods

Users pass their tidy dataframe into their chosen plotting method and specify which column names correspond visualization properties using keyword arguments. In this case, I created a grouped bar chart by specifying the ``"country"`` and ``"fruit"`` columns for the groupings and the ``"quantity"`` column
for the data value. Additionally, I passed in optional kwargs to set the bar colors and ordering.

```python
quantity_by_fruit_and_country = (tidy_data.groupby(
    ['fruit', 'country'])['quantity'].sum().reset_index())

ch = chartify.Chart(blank_labels=True, x_axis_type='categorical', y_axis_type='linear')
ch.set_title("Fruit by Country")
ch.set_subtitle("Change categorical order with 'categorical_order_by'.")
ch.plot.bar(
    data_frame=quantity_by_fruit_and_country,
    categorical_columns=['country', 'fruit'],
    numeric_column='quantity',
    color_column='country', ## optional
    categorical_order_by='labels', ## optional
    categorical_order_ascending=True ## optional
)
ch.axes.set_xaxis_tick_orientation('vertical')
ch.show()
```

<div class="bk-root" id="433d6ece-34b9-4d13-a3ab-277ef6b87924"></div>

I've very excited that Chartify exposes the Bokeh Figure object that it creates on the chart's ``.figure`` property. This means users get the wonderful functionality of a nice charting API while also being able to drop down to Bokeh-level APIs to further customize their plots. In this example, I modified a Chartify scatter plot to add a custom HoverTool and make the figure size be responsive. (You can test this by hovering over the plot and dragging the browser window larger and smaller.)

```python
from bokeh.models import HoverTool

ch = chartify.Chart(blank_labels=True, x_axis_type='datetime', y_axis_type='linear')
ch.plot.scatter(
    data_frame=tidy_data,
    x_column="date",
    y_column="total_price",
    size_column='quantity',
    color_column='fruit')

hover = HoverTool(tooltips=[
    ("Total Price (M $)", "@total_price"),
    ("Quantity Sold (M Units)", "@quantity"),
])

### access Bokeh.Figure object
ch.figure.add_tools(hover)
ch.figure.sizing_mode = 'scale_width

ch.show()
```

<div class="bk-root" id="645bb2c2-ed6a-478d-bed6-a9f133d0f676"></div>

Beyond the ``Chart.plot`` methods and accessing the Bokeh figure via ``Chart.figure``, Chartify also offers interfaces to modify plots styles, add annotations, and format the axes:

*the chartify.Chart methods*:

- Styling (.style)
- Plotting (.plot)
- Callouts (.callout)
- Axes (.axes)
- Bokeh figure (.figure)

You can views more demonstrations of these in Chartify's examples notebook [here](https://github.com/spotify/chartify/blob/master/examples/Chartify%20Tutorial.ipynb).

### Summation

Chartify offers a pleasant high-level interface for ingesting tidy data and generating a variety of visually pleasing charts, while also exposing the underlying Bokeh object for further customization. I'm excited about this addition to the Python data visualization ecosystem.

<script type="text/javascript">
  (function() {
    var fn = function() {
      Bokeh.safely(function() {
        (function(root) {
          function embed_document(root) {

          var docs_json = '{"3e781bcc-700d-45ef-830d-395978eae8a0":{"roots":{"references":[{"attributes":{"above":[{"id":"1142","type":"Title"}],"background_fill_color":{"value":"white"},"below":[{"id":"1126","type":"CategoricalAxis"},{"id":"1141","type":"Label"}],"border_fill_color":{"value":"white"},"left":[{"id":"1130","type":"LinearAxis"}],"min_border_bottom":60,"min_border_left":60,"min_border_right":60,"min_border_top":40,"outline_line_color":{"value":"white"},"plot_height":432,"plot_width":576,"renderers":[{"id":"1126","type":"CategoricalAxis"},{"id":"1129","type":"Grid"},{"id":"1130","type":"LinearAxis"},{"id":"1134","type":"Grid"},{"id":"1141","type":"Label"},{"id":"1142","type":"Title"},{"id":"1153","type":"GlyphRenderer"}],"title":{"id":"1138","type":"Title"},"toolbar":{"id":"1136","type":"Toolbar"},"x_range":{"id":"1118","type":"FactorRange"},"x_scale":{"id":"1122","type":"CategoricalScale"},"y_range":{"id":"1120","type":"DataRange1d"},"y_scale":{"id":"1124","type":"LinearScale"}},"id":"1117","subtype":"Figure","type":"Plot"},{"attributes":{"data_source":{"id":"1146","type":"ColumnDataSource"},"glyph":{"id":"1151","type":"VBar"},"hover_glyph":null,"muted_glyph":null,"nonselection_glyph":{"id":"1152","type":"VBar"},"selection_glyph":null,"view":{"id":"1154","type":"CDSView"}},"id":"1153","type":"GlyphRenderer"},{"attributes":{},"id":"1168","type":"Selection"},{"attributes":{"end":1,"factors":["BR","CA","GB","JP","US"],"palette":["#1f77b4","#ff7f0e","#2ca02c","#d62728","#9467bd"]},"id":"1147","type":"CategoricalColorMapper"},{"attributes":{"callback":null,"factor_padding":0.25,"factors":[["BR","Apple"],["BR","Banana"],["BR","Grape"],["BR","Orange"],["CA","Apple"],["CA","Banana"],["CA","Grape"],["CA","Orange"],["GB","Apple"],["GB","Banana"],["GB","Grape"],["GB","Orange"],["JP","Apple"],["JP","Banana"],["JP","Grape"],["JP","Orange"],["US","Apple"],["US","Banana"],["US","Grape"],["US","Orange"]]},"id":"1118","type":"FactorRange"},{"attributes":{"axis_label_text_color":{"value":"#666666"},"axis_label_text_font_size":{"value":"11pt"},"axis_label_text_font_style":"bold","axis_line_color":{"value":"#C0C0C0"},"formatter":{"id":"1165","type":"CategoricalTickFormatter"},"group_label_orientation":1.5707963267948966,"group_text_font_size":{"value":"11pt"},"major_label_orientation":1.5707963267948966,"major_label_text_color":{"value":"#898989"},"major_label_text_font_size":{"value":"10pt"},"major_tick_in":0,"major_tick_line_color":{"value":"#C0C0C0"},"major_tick_out":4,"minor_tick_line_color":{"value":"#C0C0C0"},"minor_tick_out":1,"plot":{"id":"1117","subtype":"Figure","type":"Plot"},"separator_line_alpha":{"value":0},"subgroup_label_orientation":1.5707963267948966,"subgroup_text_font_size":{"value":"11pt"},"ticker":{"id":"1127","type":"CategoricalTicker"}},"id":"1126","type":"CategoricalAxis"},{"attributes":{},"id":"1135","type":"SaveTool"},{"attributes":{"dimension":1,"grid_line_color":{"value":null},"plot":{"id":"1117","subtype":"Figure","type":"Plot"},"ticker":{"id":"1131","type":"BasicTicker"}},"id":"1134","type":"Grid"},{"attributes":{"format":"0,0.[000]"},"id":"1148","type":"NumeralTickFormatter"},{"attributes":{"axis_label_text_color":{"value":"#666666"},"axis_label_text_font_size":{"value":"11pt"},"axis_label_text_font_style":"bold","axis_line_color":{"value":"#C0C0C0"},"formatter":{"id":"1148","type":"NumeralTickFormatter"},"major_label_text_color":{"value":"#898989"},"major_label_text_font_size":{"value":"10pt"},"major_tick_in":0,"major_tick_line_color":{"value":"#C0C0C0"},"major_tick_out":4,"minor_tick_line_color":{"value":"#C0C0C0"},"minor_tick_out":1,"plot":{"id":"1117","subtype":"Figure","type":"Plot"},"ticker":{"id":"1131","type":"BasicTicker"}},"id":"1130","type":"LinearAxis"},{"attributes":{"plot":null,"text":"Fruit by Country","text_color":{"value":"#333333"},"text_font_size":{"value":"18pt"}},"id":"1138","type":"Title"},{"attributes":{"fill_alpha":{"value":0.1},"fill_color":{"value":"#1f77b4"},"line_alpha":{"value":0.1},"line_color":{"value":"#1f77b4"},"top":{"field":"quantity"},"width":{"value":0.9},"x":{"field":"factors"}},"id":"1152","type":"VBar"},{"attributes":{},"id":"1131","type":"BasicTicker"},{"attributes":{},"id":"1165","type":"CategoricalTickFormatter"},{"attributes":{"grid_line_color":{"value":null},"plot":{"id":"1117","subtype":"Figure","type":"Plot"},"ticker":{"id":"1127","type":"CategoricalTicker"}},"id":"1129","type":"Grid"},{"attributes":{"level":"overlay","name":"subtitle","plot":{"id":"1117","subtype":"Figure","type":"Plot"},"text":"","text_align":"right","text_color":{"value":"#898989"},"text_font_size":{"value":"10px"},"x":518.4,"x_units":"screen","y":0,"y_units":"screen"},"id":"1141","type":"Label"},{"attributes":{"callback":null,"end":null,"start":0},"id":"1120","type":"DataRange1d"},{"attributes":{"plot":{"id":"1117","subtype":"Figure","type":"Plot"},"text":"Change categorical order with &#x27;categorical_order_by&#x27;.","text_color":{"value":"#666666"},"text_font_size":{"value":"12pt"}},"id":"1142","type":"Title"},{"attributes":{},"id":"1127","type":"CategoricalTicker"},{"attributes":{"callback":null,"data":{"factors":[["BR","Apple"],["BR","Banana"],["BR","Grape"],["BR","Orange"],["CA","Apple"],["CA","Banana"],["CA","Grape"],["CA","Orange"],["GB","Apple"],["GB","Banana"],["GB","Grape"],["GB","Orange"],["JP","Apple"],["JP","Banana"],["JP","Grape"],["JP","Orange"],["US","Apple"],["US","Banana"],["US","Grape"],["US","Orange"]],"index":[0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19],"quantity":[57,30,54,74,144,222,86,207,177,113,59,97,65,232,52,75,165,479,81,409]},"name":"Series:","selected":{"id":"1168","type":"Selection"},"selection_policy":{"id":"1167","type":"UnionRenderers"}},"id":"1146","type":"ColumnDataSource"},{"attributes":{"fill_color":{"field":"factors","transform":{"id":"1147","type":"CategoricalColorMapper"}},"line_color":{"value":"white"},"top":{"field":"quantity"},"width":{"value":0.9},"x":{"field":"factors"}},"id":"1151","type":"VBar"},{"attributes":{},"id":"1122","type":"CategoricalScale"},{"attributes":{"active_drag":null,"active_inspect":"auto","active_multi":null,"active_scroll":"auto","active_tap":"auto","logo":null,"tools":[{"id":"1135","type":"SaveTool"}]},"id":"1136","type":"Toolbar"},{"attributes":{},"id":"1124","type":"LinearScale"},{"attributes":{"source":{"id":"1146","type":"ColumnDataSource"}},"id":"1154","type":"CDSView"},{"attributes":{},"id":"1167","type":"UnionRenderers"}],"root_ids":["1117"]},"title":"Bokeh Application","version":"1.0.1"}}';
          var render_items = [{"docid":"3e781bcc-700d-45ef-830d-395978eae8a0","roots":{"1117":"433d6ece-34b9-4d13-a3ab-277ef6b87924"}}];
          root.Bokeh.embed.embed_items(docs_json, render_items);

          }
          if (root.Bokeh !== undefined) {
            embed_document(root);
          } else {
            var attempts = 0;
            var timer = setInterval(function(root) {
              if (root.Bokeh !== undefined) {
                embed_document(root);
                clearInterval(timer);
              }
              attempts++;
              if (attempts > 100) {
                console.log("Bokeh: ERROR: Unable to run BokehJS code because BokehJS library is missing");
                clearInterval(timer);
              }
            }, 10, root)
          }
        })(window);
      });
    };
    if (document.readyState != "loading") fn();
    else document.addEventListener("DOMContentLoaded", fn);
  })();
</script>

<script type="text/javascript">
  (function() {
    var fn = function() {
      Bokeh.safely(function() {
        (function(root) {
          function embed_document(root) {

          var docs_json = '{"afa73682-294e-43f1-b519-fd5c3316f94e":{"roots":{"references":[{"attributes":{"above":[{"id":"1279","type":"Title"}],"background_fill_color":{"value":"white"},"below":[{"id":"1262","type":"DatetimeAxis"},{"id":"1278","type":"Label"}],"border_fill_color":{"value":"white"},"left":[{"id":"1267","type":"LinearAxis"}],"min_border_bottom":60,"min_border_left":60,"min_border_right":60,"min_border_top":40,"outline_line_color":{"value":"white"},"plot_height":540,"plot_width":960,"renderers":[{"id":"1262","type":"DatetimeAxis"},{"id":"1266","type":"Grid"},{"id":"1267","type":"LinearAxis"},{"id":"1271","type":"Grid"},{"id":"1278","type":"Label"},{"id":"1279","type":"Title"},{"id":"1307","type":"Legend"},{"id":"1288","type":"GlyphRenderer"},{"id":"1313","type":"GlyphRenderer"},{"id":"1339","type":"GlyphRenderer"},{"id":"1367","type":"GlyphRenderer"}],"sizing_mode":"scale_width","title":{"id":"1275","type":"Title"},"toolbar":{"id":"1273","type":"Toolbar"},"x_range":{"id":"1254","type":"DataRange1d"},"x_scale":{"id":"1258","type":"LinearScale"},"y_range":{"id":"1256","type":"DataRange1d"},"y_scale":{"id":"1260","type":"LinearScale"}},"id":"1253","subtype":"Figure","type":"Plot"},{"attributes":{"level":"overlay","name":"subtitle","plot":{"id":"1253","subtype":"Figure","type":"Plot"},"text":"","text_align":"right","text_color":{"value":"#898989"},"text_font_size":{"value":"10px"},"x":864.0,"x_units":"screen","y":0,"y_units":"screen"},"id":"1278","type":"Label"},{"attributes":{"months":[0,1,2,3,4,5,6,7,8,9,10,11]},"id":"1302","type":"MonthsTicker"},{"attributes":{"label":{"value":"Banana"},"renderers":[{"id":"1313","type":"GlyphRenderer"}]},"id":"1334","type":"LegendItem"},{"attributes":{},"id":"1272","type":"SaveTool"},{"attributes":{"days":[1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31]},"id":"1298","type":"DaysTicker"},{"attributes":{"mantissas":[1,2,5],"max_interval":500.0,"num_minor_ticks":0},"id":"1295","type":"AdaptiveTicker"},{"attributes":{"active_drag":null,"active_inspect":"auto","active_multi":null,"active_scroll":"auto","active_tap":"auto","logo":null,"tools":[{"id":"1272","type":"SaveTool"},{"id":"1393","type":"HoverTool"}]},"id":"1273","type":"Toolbar"},{"attributes":{"fill_color":{"value":"#d62728"},"line_color":{"value":"#d62728"},"size":{"field":"quantity","units":"screen"},"x":{"field":"date"},"y":{"field":"total_price"}},"id":"1365","type":"Scatter"},{"attributes":{"months":[0,2,4,6,8,10]},"id":"1303","type":"MonthsTicker"},{"attributes":{},"id":"1268","type":"BasicTicker"},{"attributes":{"callback":null},"id":"1256","type":"DataRange1d"},{"attributes":{"base":60,"mantissas":[1,2,5,10,15,20,30],"max_interval":1800000.0,"min_interval":1000.0,"num_minor_ticks":0},"id":"1296","type":"AdaptiveTicker"},{"attributes":{"data_source":{"id":"1363","type":"ColumnDataSource"},"glyph":{"id":"1365","type":"Scatter"},"hover_glyph":null,"muted_glyph":null,"nonselection_glyph":{"id":"1366","type":"Scatter"},"selection_glyph":null,"view":{"id":"1368","type":"CDSView"}},"id":"1367","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"value":0.1},"fill_color":{"value":"#1f77b4"},"line_alpha":{"value":0.1},"line_color":{"value":"#1f77b4"},"size":{"field":"quantity","units":"screen"},"x":{"field":"date"},"y":{"field":"total_price"}},"id":"1366","type":"Scatter"},{"attributes":{"plot":null,"text":"","text_color":{"value":"#333333"},"text_font_size":{"value":"18pt"}},"id":"1275","type":"Title"},{"attributes":{"grid_line_color":{"value":null},"plot":{"id":"1253","subtype":"Figure","type":"Plot"},"ticker":{"id":"1263","type":"DatetimeTicker"}},"id":"1266","type":"Grid"},{"attributes":{"fill_alpha":{"value":0.1},"fill_color":{"value":"#1f77b4"},"line_alpha":{"value":0.1},"line_color":{"value":"#1f77b4"},"size":{"field":"quantity","units":"screen"},"x":{"field":"date"},"y":{"field":"total_price"}},"id":"1338","type":"Scatter"},{"attributes":{"base":24,"mantissas":[1,2,4,6,8,12],"max_interval":43200000.0,"min_interval":3600000.0,"num_minor_ticks":0},"id":"1297","type":"AdaptiveTicker"},{"attributes":{},"id":"1258","type":"LinearScale"},{"attributes":{"callback":null,"data":{"date":{"__ndarray__":"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","dtype":"float64","shape":[222]},"fruit":["Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape","Grape"],"index":[3,9,10,18,24,36,40,41,47,48,58,61,62,72,84,90,94,95,96,110,111,113,117,118,123,126,127,129,131,135,144,147,149,151,152,154,155,159,161,163,164,171,179,181,186,198,199,206,216,221,225,229,236,239,241,245,249,252,253,254,259,260,261,263,266,279,280,291,292,306,308,315,316,318,320,322,330,337,343,348,350,368,369,375,381,382,383,387,389,395,398,402,405,406,407,413,421,425,426,429,447,451,453,454,457,465,467,470,471,474,478,487,488,491,500,501,508,515,517,518,519,523,529,533,534,536,538,552,562,578,580,591,593,594,596,606,620,623,630,648,649,654,657,661,665,669,672,683,687,694,706,708,712,715,717,718,721,730,739,747,748,750,755,756,757,760,764,771,779,784,788,815,818,821,824,825,829,831,844,847,848,851,853,854,858,861,864,865,866,876,877,885,894,902,917,920,921,924,925,927,928,935,936,937,940,942,950,952,953,954,958,960,961,966,968,972,980,984,985,986,992,999],"quantity":[2,2,1,3,1,3,2,2,1,1,2,1,2,1,1,2,3,1,1,2,1,4,1,1,2,1,1,2,2,1,2,3,1,2,2,1,1,3,2,1,2,1,1,1,2,1,1,1,2,2,1,1,2,1,3,1,1,1,1,1,2,1,2,1,2,2,2,2,1,1,1,3,1,1,2,1,2,2,5,1,1,1,1,1,1,1,1,1,2,1,3,2,3,2,1,1,2,1,1,1,1,3,2,2,1,2,2,2,1,2,1,1,1,3,1,2,1,1,1,1,1,3,2,1,1,1,1,2,1,1,2,1,1,1,1,1,1,1,1,1,1,4,1,1,2,2,2,2,3,2,2,1,1,1,1,2,2,1,2,1,1,3,1,1,1,2,1,1,1,2,2,1,1,2,2,1,1,1,1,1,1,2,2,1,1,1,1,1,1,2,2,1,1,1,1,1,1,2,1,2,1,3,2,1,1,1,2,1,2,1,2,1,2,1,1,2,1,2,1,1,1,1],"total_price":{"__ndarray__":"XwKsJ+O4EUB/xvLwXHQNQKQxOz2B2QBAPbS1NVrjFEB6SvMbHLwCQELM19odhxlAW3wwzlkDDUAB5b6E9C4QQA81XmlvZ/0/5cdoI4vOAEBN4dYXohAQQNE7+xjaowBAl3NYl/ZlDECdAXB7HyIBQP83ms4+6gBAzSfot2fPDEAxrocYTxEWQB51qqCXIQJAya6gJG5UAUB46QDe2ioPQJ3cc1/rnP4/23FIa6FUIEBIzVRnV4D+PxFatC77TwFAtCIOMlAiDUB2yERYavoAQMVT5SH89wBAMPRiclY6EUATL79iEM4OQCIAAkDdEf0/eZmfzkOIEEDBYL1/7xETQIFvRkzgaP8/j7cS9uvuB0DZbvZIOP0NQO6xWNNSMAJAYzXs/3sFAEC4OIzZPmQYQGGHCxgkwRBAG1mC1I6K+z+zVCMTXGEJQJddzBBTTAJAVi879kcO+j8hJkLeM6EAQMzS1+YdOw9AzJv4v6hbAUAby2h5uuP6P58hRoysZQJApbrmT6j5EEAx1IRaJZQJQMaB2nq//wJAIKZ2ocRPAUA55F/RVqQPQJK2IthqyQJASM5j4EVpE0DGkoViCAMAQGQwdkwsEgFAdVnSQW7hAEAFOBMYvxIDQAdCnNqW2QFAdXptYHcGDEDq2KaE6aMAQLsMNnwd8A9A0d9XOLBVAEBOMQvsjB4PQPg1cXrbMQ1AXXIAEFR3EEAZkiuHuAARQN7E0WJWA/8/+x5x5YQ8AUD5bXOZ0kEBQFgBGnUd5BRAb00K+Z9P/T97Vrn3SUUCQLL5nVMkow5AejeidVqeAEADcteCyAYMQJkePxO6ngtADPaPxvRYIEAV6uuuaQn+P5QY2XyZmvo/oohpT95zAEC2JtDoOasBQLytwkaKrABADbgX0b0m+T+mySJyOFH/P2GHpbymIgRAhd2K3pEgAEC8lnSoPpEPQE/mHIYVFP4/kbDasDaOE0DrCvp/G+IJQCrdjBgoQRRACrhrFIIKDUBLoIrQwG//P0Gkve7C0ABAz+LgXslFEEAG01Mz3ocCQPWiJg467v4/nJjMVm/iAkBXaUd2WPMCQFe0iXObUhVA8FE8m8SyDkATG8Q8vwkQQAi7Os8gr/o/n+l7vSQcEEAdXPj6M8EQQPI6NzgeiQ1ALiBk1xHtAUCWgLYQP8UPQJR59BRM2/8/WEq9EVq9/j+DAnVWP9//P6pD/ZqYuBJAG7GBhiPi/j+b3uKCPC8OQGAFZubu8ABAHJOFo0MSAEAS34mURxUCQDEHXKTU5gBAnnvUypR6+j9GC96CYlUWQL3T2fPT0QxACOuRKgfg+z+Sod5KbfcAQF323ewz6P8/lGUuNjgO/z8WD+Sl3+UOQInsM8hR1gBABx2Y3DyI/z9cfsrqJZgRQLHtw4TXWABAFOyby6gcAkDBynRLO3sCQGIOJZg79AFADEY45dBH/D+0yodPyRYAQAtJ1OQhIAFALrH51RkXA0B17V0yAR4BQOoZyn7gswJAATnzL0IiGkDY/9EGSM/+P2N6apXlHfw/32ZXGwd8EEAHKx0f1XsPQOFC9zb36hBAVj+uJg3yEEBZuh4P09AaQFdxPCrQ2xBAOXElC/8jD0AgclXKULj+P6xgTdCeC/w/VYECs+MeAEAunEkm2PD6PwxXbXVi6RBA5I5L6bYwDkBPyMNoKqEBQIVjfGwD8wxAZp1hCfu8AUBKm2ikFJX+P7zcrUmF1xdA/DZfNYGp/T9hnI4dzlr/P2V77m1G0AFArgZBGLHKD0AP5iwJkWb8P3rNi+gxmgJAT1N0tmFVAEAMT6eFvmIPQIgYu3Braw5AkKJUR7/NAkBVdb4ODpr/P+rX4Wu2Bw5ACOro3IOcCkDwaDQaD+72P9KoFXUg7v4/cQBz93tNAEDOiv+UJ1z7Px8gygXcvv8/MGMKgyllAECM/Pgk++UQQG0PVmfrDA9AuBFAe9UJAEA6RH0I4UoCQGHK8r1kuAFAHARA5jadAEChmGs7cs8AQMKAOw4HQQBADONkcTqkDEDTTLc72osQQK240LfIiQFAXh10QK0sA0BC2mMvYHIDQCu+RpPX7wFAUeEl4NoLAEBEG3ncXcAAQJ5wsRneVQtAI35g68xc/z/V3zh6YagRQEFs2eRmowBAQ+Xvvcv/FUCHMTykhf8QQGmqgs4oh/4/7hB7ItBCAkCAzNft8GEBQKzCpiTepxFAbhmfpDyHAEB4neTaNHsQQI/Z+Z7MDwJArwxFTqJFEUDBXLEWWZYBQP90pWUk9g9AkHXalu/3AUCZft08rkkBQDgWnoAYbxJATfg+7Xc1/j8owp55OWYPQH8EPjSvUv0/iDE8AnAlAEAYOJp/wfn/P49bzP1uCwBA","dtype":"float64","shape":[222]}},"name":"Series:Grape","selected":{"id":"1391","type":"Selection"},"selection_policy":{"id":"1390","type":"UnionRenderers"}},"id":"1335","type":"ColumnDataSource"},{"attributes":{"source":{"id":"1363","type":"ColumnDataSource"}},"id":"1368","type":"CDSView"},{"attributes":{},"id":"1432","type":"UnionRenderers"},{"attributes":{},"id":"1390","type":"UnionRenderers"},{"attributes":{"source":{"id":"1284","type":"ColumnDataSource"}},"id":"1289","type":"CDSView"},{"attributes":{},"id":"1306","type":"YearsTicker"},{"attributes":{"fill_alpha":{"value":0.1},"fill_color":{"value":"#1f77b4"},"line_alpha":{"value":0.1},"line_color":{"value":"#1f77b4"},"size":{"field":"quantity","units":"screen"},"x":{"field":"date"},"y":{"field":"total_price"}},"id":"1312","type":"Scatter"},{"attributes":{},"id":"1391","type":"Selection"},{"attributes":{"dimension":1,"grid_line_color":{"value":null},"plot":{"id":"1253","subtype":"Figure","type":"Plot"},"ticker":{"id":"1268","type":"BasicTicker"}},"id":"1271","type":"Grid"},{"attributes":{"days":[1,8,15,22]},"id":"1300","type":"DaysTicker"},{"attributes":{"label":{"value":"Orange"},"renderers":[{"id":"1367","type":"GlyphRenderer"}]},"id":"1392","type":"LegendItem"},{"attributes":{"fill_color":{"value":"#2ca02c"},"line_color":{"value":"#2ca02c"},"size":{"field":"quantity","units":"screen"},"x":{"field":"date"},"y":{"field":"total_price"}},"id":"1337","type":"Scatter"},{"attributes":{},"id":"1333","type":"Selection"},{"attributes":{"axis_label_text_color":{"value":"#666666"},"axis_label_text_font_size":{"value":"11pt"},"axis_label_text_font_style":"bold","axis_line_color":{"value":"#C0C0C0"},"formatter":{"id":"1293","type":"DatetimeTickFormatter"},"major_label_text_color":{"value":"#898989"},"major_label_text_font_size":{"value":"10pt"},"major_tick_in":0,"major_tick_line_color":{"value":"#C0C0C0"},"major_tick_out":4,"minor_tick_line_color":{"value":"#C0C0C0"},"minor_tick_out":1,"plot":{"id":"1253","subtype":"Figure","type":"Plot"},"ticker":{"id":"1263","type":"DatetimeTicker"}},"id":"1262","type":"DatetimeAxis"},{"attributes":{"days":[1,4,7,10,13,16,19,22,25,28]},"id":"1299","type":"DaysTicker"},{"attributes":{},"id":"1260","type":"LinearScale"},{"attributes":{"data_source":{"id":"1335","type":"ColumnDataSource"},"glyph":{"id":"1337","type":"Scatter"},"hover_glyph":null,"muted_glyph":null,"nonselection_glyph":{"id":"1338","type":"Scatter"},"selection_glyph":null,"view":{"id":"1340","type":"CDSView"}},"id":"1339","type":"GlyphRenderer"},{"attributes":{},"id":"1433","type":"Selection"},{"attributes":{"callback":null},"id":"1254","type":"DataRange1d"},{"attributes":{"source":{"id":"1335","type":"ColumnDataSource"}},"id":"1340","type":"CDSView"},{"attributes":{"label":{"value":"Apple"},"renderers":[{"id":"1288","type":"GlyphRenderer"}]},"id":"1308","type":"LegendItem"},{"attributes":{"plot":{"id":"1253","subtype":"Figure","type":"Plot"},"text":"","text_color":{"value":"#666666"},"text_font_size":{"value":"12pt"}},"id":"1279","type":"Title"},{"attributes":{"fill_color":{"value":"#ff7f0e"},"line_color":{"value":"#ff7f0e"},"size":{"field":"quantity","units":"screen"},"x":{"field":"date"},"y":{"field":"total_price"}},"id":"1311","type":"Scatter"},{"attributes":{"callback":null,"data":{"date":{"__ndarray__":"AACAXEgBdkIAAABpveF1QgAAgLGL8HVCAACAJ9LodUIAAMCnX/V1QgAAgLBVvnVCAACAT7gHdkIAAEBFcfN1QgAAwM4P4nVCAABAwpoBdkIAAEDPKvt1QgAAgH1LCnZCAAAA7JPTdUIAAIDYO911QgAAQDerx3VCAACAadj6dUIAAEAPxah1QgAAQEVx83VCAACAQfLbdUIAAMCGXOx1QgAAQPAtBHZCAABA450KdkIAAMBkI7F1QgAAAPN26XVCAAAABrTGdUIAAAAUevJ1QgAAQCRu6nVCAADAk+zldUIAAICJpdF1QgAAQF5btHVCAACAnRjhdUIAAADezad1QgAAwICvCHZCAACAgsK7dUIAAIANsvV1QgAAQD6O3XVCAACA+Ai0dUIAAABOZ7x1QgAAwBAW9HVCAAAAkaMAdkIAAID4CLR1QgAAgFVl63VCAADAZVnjdUIAAMCMCdB1QgAAACEK7HVCAAAApODddUIAAIBPuAd2QgAAAOyT03VCAADAf3nWdUIAAIB8Fdh1QgAAQPAtBHZCAADAms/7dUIAAEDIR+V1QgAAQDH+43VCAABAs56jdUIAAAAumuV1QgAAAJBtznVCAADA6WUHdkIAAAB9MPF1QgAAAIQTB3ZCAADAjAnQdUIAAMBLOfB1QgAAQMFkz3VCAAAAVoAEdkIAAMAkiQN2QgAAACjtAXZCAADAHabtdUIAAACQbc51QgAAgEefv3VCAACAynWxdUIAAADtyQV2QgAAwCSJA3ZCAAAAAAfjdUIAAIAaQu91QgAAgLYConVCAADAwX/odUIAAACXUOR1QgAAgBpC73VCAABAtNTVdUIAAMC07+51QgAAQF5btHVCAAAAW/e1dUIAAMCnX/V1QgAAgMur43VCAADA1fL3dUIAAMDigvF1QgAAwJrP+3VCAABAoGHGdUIAAAD6Wf91QgAAAH0w8XVCAADA/KLkdUIAAAA0R8l1QgAAAN7Np3VCAADAbDz5dUIAAIBcSAF2QgAAQCRu6nVCAABAtNTVdUIAAACd/cd1QgAAQMKaAXZCAAAA3wPadUIAAEC1Cgh2QgAAAPN26XVCAADAMRn9dUIAAICdGOF1QgAAAEeEpnVCAABAN6vHdUIAAADZVvZ1QgAAwEoDvnVCAABAN6vHdUIAAIBB8tt1QgAAwIwJ0HVCAAAAB+r4dUIAAAAtZLN1QgAAwPyi5HVCAADAzg/idUIAAEAWqL51QgAAAFQUoHVCAAAA+SPNdUIAAMBEVtp1QgAAAPpZ/3VCAADA7xLrdUIAAED8h8t1QgAAgLhuBnZCAAAAcKD3dUIAAIC4bgZ2QgAAQAkYxXVCAAAAvgDRdUIAAEBeW7R1QgAAQExUCXZCAACAxMjNdUIAAEDcuvR1QgAAgOyu7HVCAADA7xLrdUIAAIBvhd51QgAAwD6p9nVCAAAAfTDxdUIAAEAewQZ2QgAAALKmCXZCAACAACL8dUIAAACKwOp1QgAAQB7BBnZCAADAupzSdUIAAIDe6MB1QgAAgPgItHVCAADA/KLkdUIAAECGQdN1QgAAABon1nVCAACAuG4GdkIAAEBMVAl2QgAAAPN26XVCAACAsYvwdUIAAMCNPwJ2QgAAABtdCHZCAADAZCOxdUIAAEAdi9R1QgAAQPAtBHZCAACAISUFdkIAAEByzsN1QgAAgKPFxHVCAADA9b/OdUIAAADzdul1QgAAgC1/zHVCAABAhQuhdUIAAEA+jt11QgAAAHCg93VCAADAeczydUIAAACEEwd2QgAAgC61/nVCAABAHsEGdkIAAIDRWMd1QgAAgNFYx3VCAACAT7gHdkIAAEBfkeZ1QgAAQNy69HVCAABAlAf/dUIAAIBvhd51QgAAAPkjzXVCAACAT7gHdkIAAIA0YuJ1QgAAQHLOw3VCAADAI1PRdUIAAIANsvV1QgAAALFw13VCAACAtzjUdUIAAIC95bd1QgAAgCElBXZCAACAE1/ZdUIAAECUB/91QgAAAFaABHZCAABAMf7jdUIAAMBYyel1QgAAwAOG+nVCAADAwEm2dUIAAMDigvF1QgAAgC1/zHVCAACAACL8dUIAAECuJ/J1QgAAwFeTt3VCAAAAnf3HdUIAAIAGz991QgAAgOXL1nVCAADAjAnQdUIAAMDpZQd2QgAAgJX/mHVCAADAawbHdUIAAEAPxah1QgAAwDfG4HVCAABACRjFdUIAAMAWw9d1QgAAgFxIAXZCAACAuG4GdkIAAEDcuvR1QgAAADsq33VCAACAxMjNdUIAAMC70gR2QgAAgE+4B3ZCAADAhlzsdUIAAAAo7QF2QgAAwNXy93VCAADAEBb0dUIAAMBeds11QgAAwBf5CXZCAACA8lvQdUIAAECHdwV2QgAAwOllB3ZCAACAljXLdUIAAEDPKvt1QgAAQCtRAHZCAADAms/7dUIAAAB8+r51QgAAgG5PrHVCAAAAkG3OdUIAAEBMVAl2QgAAgLhuBnZCAACA8lvQdUIAAMBsPPl1QgAAAGLay3VCAADAyGL+dUIAAAA7Kt91QgAAQB7BBnZCAADA4oLxdUIAAECgYcZ1QgAAgC61/nVCAAAAOvSsdUIAAIC4bgZ2QgAAwICvCHZCAACAIO/SdUIAAMDoL9V1QgAAQLUKCHZCAAAAq8PzdUIAAEAkbup1QgAAQKGX+HVCAAAAB+r4dUIAAEDcuvR1QgAAgNg73XVCAACA+T7mdUIAAIDfHvN1QgAAQMKaAXZCAACA2DvddUIAAIDEyM11QgAAAJ4z+nVCAABA1dfedUIAAAAnt891QgAAABtdCHZCAABAu7frdUIAAMD1v851QgAAQId3BXZC","dtype":"float64","shape":[267]},"fruit":["Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange","Orange"],"index":[12,14,15,17,20,22,25,26,31,33,34,38,45,49,50,51,56,57,64,66,67,75,76,77,80,81,88,93,98,99,100,101,104,105,106,119,121,124,125,130,134,139,141,143,146,153,158,160,165,169,170,172,173,193,194,205,211,213,214,220,227,228,235,240,243,248,251,255,256,265,273,278,282,284,288,295,296,297,299,302,304,309,312,313,314,317,319,323,326,327,340,341,342,344,346,352,354,362,363,365,371,379,392,394,397,399,400,403,404,418,423,427,437,439,443,445,452,455,461,464,469,483,484,489,490,492,493,494,495,496,497,505,506,507,510,513,514,516,520,524,530,532,540,541,555,556,563,564,565,569,573,584,585,588,592,603,610,618,621,626,629,634,636,644,645,647,651,652,655,660,662,663,667,671,676,678,684,690,692,695,697,699,701,705,709,714,716,722,723,724,729,736,740,742,743,744,749,751,754,759,762,766,768,769,773,777,781,782,786,789,790,791,793,800,806,808,809,812,813,817,819,823,828,832,833,835,838,840,846,850,855,856,860,868,869,874,880,881,888,892,895,896,897,908,916,918,919,923,930,932,933,938,941,945,946,951,955,962,976,977,979,987,989,991,995,997,998],"quantity":[5,2,4,4,6,2,4,2,4,2,2,2,2,6,4,7,5,2,2,6,3,4,4,4,3,2,5,4,4,5,2,3,3,2,5,1,3,1,1,3,2,3,5,3,2,5,6,3,5,3,1,2,5,7,2,3,4,3,2,5,4,2,2,6,2,1,2,2,2,2,4,2,2,4,3,3,3,2,4,4,5,6,2,3,3,1,3,1,2,1,3,3,3,1,5,4,2,3,5,3,3,1,5,2,5,3,1,6,1,3,3,2,4,3,2,3,4,5,5,7,3,2,2,3,3,2,3,3,4,2,3,4,4,3,3,2,4,2,4,3,4,5,3,4,5,2,4,4,4,2,2,4,3,4,2,4,3,2,4,2,2,5,2,2,6,6,1,4,2,4,4,4,1,5,4,3,3,3,2,4,4,2,3,1,4,2,3,4,4,6,5,1,4,2,4,2,4,3,4,4,4,4,2,2,3,2,2,3,7,4,4,5,3,3,1,1,3,3,3,2,4,4,4,2,4,2,2,2,2,2,4,4,5,3,4,6,3,1,3,2,3,4,4,3,4,3,3,2,3,3,4,5,5,3,5,1,2,3,3,6,3,3,5,2,5,2,1],"total_price":{"__ndarray__":"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","dtype":"float64","shape":[267]}},"name":"Series:Orange","selected":{"id":"1433","type":"Selection"},"selection_policy":{"id":"1432","type":"UnionRenderers"}},"id":"1363","type":"ColumnDataSource"},{"attributes":{"items":[{"id":"1308","type":"LegendItem"},{"id":"1334","type":"LegendItem"},{"id":"1362","type":"LegendItem"},{"id":"1392","type":"LegendItem"}],"location":"top_left","orientation":"horizontal","plot":{"id":"1253","subtype":"Figure","type":"Plot"}},"id":"1307","type":"Legend"},{"attributes":{"months":[0,4,8]},"id":"1304","type":"MonthsTicker"},{"attributes":{"data_source":{"id":"1309","type":"ColumnDataSource"},"glyph":{"id":"1311","type":"Scatter"},"hover_glyph":null,"muted_glyph":null,"nonselection_glyph":{"id":"1312","type":"Scatter"},"selection_glyph":null,"view":{"id":"1314","type":"CDSView"}},"id":"1313","type":"GlyphRenderer"},{"attributes":{},"id":"1293","type":"DatetimeTickFormatter"},{"attributes":{"months":[0,6]},"id":"1305","type":"MonthsTicker"},{"attributes":{},"id":"1360","type":"UnionRenderers"},{"attributes":{"label":{"value":"Grape"},"renderers":[{"id":"1339","type":"GlyphRenderer"}]},"id":"1362","type":"LegendItem"},{"attributes":{},"id":"1361","type":"Selection"},{"attributes":{"callback":null,"data":{"date":{"__ndarray__":"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","dtype":"float64","shape":[224]},"fruit":["Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple","Apple"],"index":[5,7,8,11,19,21,23,30,32,37,43,46,53,59,63,69,82,86,102,107,108,112,114,116,145,148,150,157,162,168,175,176,177,180,182,183,185,189,191,197,200,202,204,208,212,215,223,224,226,231,234,237,258,264,267,268,271,272,275,281,283,290,298,300,303,305,311,321,324,325,329,331,333,335,336,338,345,347,355,356,358,361,385,388,391,393,408,409,410,416,419,424,428,431,435,436,438,444,446,448,449,456,458,459,462,468,480,485,498,502,504,526,535,539,542,543,545,547,548,549,550,551,553,554,557,559,560,567,571,575,579,586,589,595,597,599,601,605,607,609,612,613,622,631,633,635,637,638,643,646,653,659,668,674,677,680,681,682,685,691,693,696,703,707,710,720,725,726,727,728,731,732,735,737,738,741,745,746,758,770,778,785,792,796,798,802,804,805,807,810,820,822,826,827,834,837,839,841,842,849,852,857,859,867,870,875,878,883,884,890,893,901,907,914,929,934,943,956,970,978,982,983,988,993],"quantity":[2,4,4,2,1,3,5,3,2,1,4,2,2,5,3,2,2,2,3,3,4,2,3,1,3,3,3,4,1,4,1,2,3,4,2,4,3,2,5,2,2,4,2,4,3,2,5,3,4,4,3,2,5,4,5,2,2,6,1,1,3,2,2,2,5,1,4,4,2,4,3,4,3,1,1,3,5,4,3,2,1,3,3,2,3,3,2,6,1,2,3,1,4,4,3,2,2,2,1,3,4,1,1,2,2,3,2,3,3,1,3,2,4,1,1,2,3,2,2,3,3,2,4,2,5,6,2,5,4,2,3,2,3,2,3,2,5,2,1,3,1,4,5,2,2,3,3,3,1,3,1,9,3,2,2,2,2,1,3,2,3,2,4,2,1,2,3,2,3,3,4,4,2,3,1,5,2,1,3,2,3,2,2,3,2,1,3,3,3,1,1,2,5,3,2,2,2,3,3,4,3,3,2,2,1,1,2,2,5,3,3,3,2,2,1,2,3,4,4,2,3,3,2,4],"total_price":{"__ndarray__":"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","dtype":"float64","shape":[224]}},"name":"Series:Apple","selected":{"id":"1333","type":"Selection"},"selection_policy":{"id":"1332","type":"UnionRenderers"}},"id":"1284","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"1284","type":"ColumnDataSource"},"glyph":{"id":"1286","type":"Scatter"},"hover_glyph":null,"muted_glyph":null,"nonselection_glyph":{"id":"1287","type":"Scatter"},"selection_glyph":null,"view":{"id":"1289","type":"CDSView"}},"id":"1288","type":"GlyphRenderer"},{"attributes":{"num_minor_ticks":5,"tickers":[{"id":"1295","type":"AdaptiveTicker"},{"id":"1296","type":"AdaptiveTicker"},{"id":"1297","type":"AdaptiveTicker"},{"id":"1298","type":"DaysTicker"},{"id":"1299","type":"DaysTicker"},{"id":"1300","type":"DaysTicker"},{"id":"1301","type":"DaysTicker"},{"id":"1302","type":"MonthsTicker"},{"id":"1303","type":"MonthsTicker"},{"id":"1304","type":"MonthsTicker"},{"id":"1305","type":"MonthsTicker"},{"id":"1306","type":"YearsTicker"}]},"id":"1263","type":"DatetimeTicker"},{"attributes":{"callback":null,"data":{"date":{"__ndarray__":"AAAAq8PzdUIAAABvasV1QgAAQNuEwnVCAAAAvzYDdkIAAIBi9eR1QgAAwMhi/nVCAADAD+DBdUIAAABB18J1QgAAgN8e83VCAAAAuFPtdUIAAABPne51QgAAwM3Zr3VCAADAHHC7dUIAAMA2kK51QgAAQHmx2XVCAAAAxePmdUIAAIC+G+p1QgAAgDRi4nVCAAAAzMb8dUIAAEAQ+9p1QgAAQId3BXZCAACABZmtdUIAAIDsrux1QgAAgNg73XVCAACAQLypdUIAAICD+O11QgAAAL82A3ZCAACAO0X4dUIAAICQiOd1QgAAQNuEwnVCAABARDvBdUIAAIDSjvl1QgAAQGwh4HVCAAAAXC3odUIAAACqjcF1QgAAgLc41HVCAABA3Lr0dUIAAABB18J1QgAAgOyu7HVCAAAA7ckFdkIAAIBuT6x1QgAAwERW2nVCAAAAhBMHdkIAAMAqNud1QgAAAFaABHZCAAAAvzYDdkIAAICPUrV1QgAAwKdf9XVCAACAfBXYdUIAAADLkMp1QgAAgKT79nVCAABArfG/dUIAAMDN2a91QgAAgLGL8HVCAADA4oLxdUIAAEDCmgF2QgAAQFnkAnZCAACAdmj0dUIAAMD7bLJ1QgAAgPORAnZCAAAAzMb8dUIAAMCfRq11QgAAAMXj5nVCAAAAVoAEdkIAAIA0YuJ1QgAAgMTIzXVCAACAYb+ydUIAAIAzLLB1QgAAgFxIAXZCAADA4oLxdUIAAEADa+F1QgAAgMX+/3VCAAAAxePmdUIAAEAkbup1QgAAQK4n8nVCAACAitsDdkIAAADYIMR1QgAAQICU73VCAADAAlDIdUIAAEDPKvt1QgAAgBpC73VCAACAsYvwdUIAAIDF/v91QgAAQD1Yq3VCAAAANEfJdUIAAED9vf11QgAAgE6C1XVCAABAHYvUdUIAAIC4bgZ2QgAAwPW/znVCAACALrX+dUIAAEBfkeZ1QgAAwJrP+3VCAADAcuncdUIAAMBfrP91QgAAgMTIzXVCAACAdmj0dUIAAAA0R8l1QgAAAFaABHZCAABA4mfYdUIAAMC70gR2QgAAgI9StXVCAAAAQg31dUIAAABB18J1QgAAwHnM8nVCAABArifydUIAAIAaQu91QgAAQM70yHVCAADAzg/idUIAAIB9Swp2QgAAQD6O3XVCAACAfUsKdkIAAMDUvMV1QgAAwHnM8nVCAACAfUsKdkIAAMD8ouR1QgAAQId3BXZCAABAEPvadUIAAMCZmcl1QgAAAFaABHZCAAAAcKD3dUIAAAAumuV1QgAAgJCI53VCAADAUhwGdkIAAMAqNud1QgAAwPb1AHZCAACAGkLvdUIAAMD1v851QgAAgEjV8XVCAACAACL8dUIAAMCMCdB1QgAAQICU73VCAAAAfTDxdUIAAECHdwV2QgAAQLu363VCAACAGkLvdUIAAAAbXQh2QgAAgG+F3nVCAABA/b39dUIAAEBFcfN1QgAAQNXX3nVCAACABs/fdUIAAEAx/uN1QgAAwK0M2XVCAAAA5ubvdUIAAADLkMp1QgAAwO8S63VCAABAcwT2dUIAAEBSAe11QgAAAHZN23VCAAAA83bpdUIAAMCL0511QgAAQGZ0/HVCAAAAIQrsdUIAAMDIYv51QgAAwCkAtXVCAAAAVoAEdkIAAIDmAQl2QgAAwPb1AHZCAACAitsDdkIAAABjEP51QgAAgL4b6nVCAACAitsDdkIAAMBy6dx1QgAAAJdQ5HVCAACAfUsKdkIAAMCtDNl1QgAAgL3lt3VCAADAyGL+dUIAAEAewQZ2QgAAQKBhxnVCAADACP2rdUIAAMB/edZ1QgAAQEVx83VCAADAms/7dUIAAIBcSAF2QgAAQD6O3XVCAACA5gEJdkIAAEDwLQR2QgAAwCSJA3ZCAAAAFHrydUIAAADLkMp1QgAAAN8D2nVCAADAZVnjdUIAAICD+O11QgAAAL82A3ZCAAAAFHrydUIAAMBxs6p1QgAAgB+5oHVCAADAupzSdUIAAIDfHvN1QgAAQB2L1HVCAACAiaXRdUIAAIBHn791QgAAAIQTB3ZCAAAAVUrSdUIAAICQiOd1QgAAQEVx83VCAAAAFHrydUIAAIDKdbF1QgAAgGL15HVCAAAAkG3OdUIAAMB4lsB1QgAAQPAtBHZCAADAUebTdUIAAECznqN1QgAAwIZc7HVCAADAMRn9dUIAAMD1v851QgAAAOWwvXVCAACA7K7sdUIAAIAutf51QgAAQIZB03VCAABAoGHGdUIAAAANl9x1QgAAwKdf9XVCAADAUebTdUIAAICxi/B1QgAAwCSJA3ZCAACAACL8dUIAAIBcSAF2QgAAACjtAXZCAACAQfLbdUIAAEDwLQR2QgAAQBfe8HVCAADAEBb0dUIAAICxi/B1QgAAwICvCHZCAADApinDdUIAAMB/edZ1QgAAANlW9nVCAABACRjFdUIAAED9vf11QgAAwJmZyXVCAAAAVoAEdkIAAEAkbup1QgAAAA2X3HVCAAAAhBMHdkIAAIA7Rfh1QgAAgG+F3nVCAACAT7gHdkIAAMB/edZ1QgAAwOKC8XVCAADAF/kJdkIAAAAMYap1QgAAQMFkz3VCAAAAXC3odUIAAACeM/p1QgAAQI0k6XVCAADAF/kJdkIAAEAewQZ2QgAAwIwJ0HVCAACAtzjUdUIAAADtyQV2QgAAwI0/AnZCAACAqqjadUIAAEDjnQp2QgAAgDRi4nVCAACAfUsKdkIAAIBi9eR1QgAAAJ4z+nVCAACA3x7zdUIAAMBsPPl1QgAAwFIcBnZCAABArifydUIAAAA69Kx1QgAAQOgUvHVCAADAEBb0dUIAAIBcSAF2QgAAwDfG4HVCAABAX5HmdUIAAEAewQZ2QgAAgKqo2nVCAAAAkaMAdkIAAECUB/91QgAAwOFMv3VCAADA7xLrdUIAAEBmdPx1QgAAQExUCXZCAADArQzZdUIAAECUB/91QgAAwLvSBHZCAADAULChdUIAAIBi9eR1QgAAgHZo9HVCAACAgsK7dUIAAECuJ/J1Qg==","dtype":"float64","shape":[287]},"fruit":["Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana","Banana"],"index":[0,1,2,4,6,13,16,27,28,29,35,39,42,44,52,54,55,60,65,68,70,71,73,74,78,79,83,85,87,89,91,92,97,103,109,115,120,122,128,132,133,136,137,138,140,142,156,166,167,174,178,184,187,188,190,192,195,196,201,203,207,209,210,217,218,219,222,230,232,233,238,242,244,246,247,250,257,262,269,270,274,276,277,285,286,287,289,293,294,301,307,310,328,332,334,339,349,351,353,357,359,360,364,366,367,370,372,373,374,376,377,378,380,384,386,390,396,401,411,412,414,415,417,420,422,430,432,433,434,440,441,442,450,460,463,466,472,473,475,476,477,479,481,482,486,499,503,509,511,512,521,522,525,527,528,531,537,544,546,558,561,566,568,570,572,574,576,577,581,582,583,587,590,598,600,602,604,608,611,614,615,616,617,619,624,625,627,628,632,639,640,641,642,650,656,658,664,666,670,673,675,679,686,688,689,698,700,702,704,711,713,719,733,734,752,753,761,763,765,767,772,774,775,776,780,783,787,794,795,797,799,801,803,811,814,816,830,836,843,845,862,863,871,872,873,879,882,886,887,889,891,898,899,900,903,904,905,906,909,910,911,912,913,915,922,926,931,939,944,947,948,949,957,959,963,964,965,967,969,971,973,974,975,981,990,994,996],"quantity":[4,4,4,5,7,3,4,6,3,3,5,6,4,3,4,4,3,3,2,4,4,6,3,3,4,4,5,4,2,6,9,5,2,5,7,6,2,4,2,7,5,2,7,4,2,3,4,3,6,3,5,5,3,3,5,5,1,2,1,4,3,2,4,3,5,5,7,5,3,7,5,4,4,4,3,4,2,5,9,2,5,6,1,6,3,4,9,3,2,6,3,2,4,3,2,4,2,3,2,5,2,2,4,4,4,4,4,5,4,2,2,3,4,8,4,3,3,3,4,6,1,2,2,3,4,5,5,3,3,1,3,2,1,6,3,3,2,1,6,2,2,4,2,2,3,2,2,4,5,4,3,6,3,5,3,3,3,4,4,5,5,2,6,3,1,4,3,3,2,2,3,3,9,7,4,4,4,2,4,5,4,2,5,2,2,2,4,4,3,2,3,6,2,3,8,3,4,3,2,3,2,3,3,2,3,5,3,10,6,1,5,6,2,5,3,2,3,6,5,5,3,3,7,5,4,3,4,4,2,4,4,6,4,2,5,1,2,4,5,5,3,1,3,4,6,3,2,2,2,6,1,1,6,3,3,5,8,3,5,4,3,4,4,5,3,7,7,3,2,4,8,1,2,6,2,2,2,3,1,7,1,3,7,5,1,4,1],"total_price":{"__ndarray__":"cJPvDgJw8z/YTSWZUkPwPwvIXdNPMfE/epyynb+q+D+SKm+PnUT/P/t1ikthDuk/4CQNpX9E8T+8u7JWYJX3P/B0bFk1xOY/yOcNL7Bg7z9ogY6x89T1P3CW9mlqPPY/8s4mNdux8D+5/ItdFBfqP7QrhJ30ue8/8yOgQZfS7j84nzKYd+7oP5Dn/2W5ROs/Sn/k0/jB3z/Fl76TmLjwP1eyRuMiDO0/Lgctv054+z+cf7m2sCHpP0lJWR777OQ/sz6xTUCm8T9DnXxlkQruP+c1yzZ3tPM/8oMeW8Xt7j/RWrsPHJ/bP/riPqMV+fc/cLyrK20oBEDz3WPp0kb0P18GEhgDEuM/gKoBTg+L8j9evAiv3DkAQIww6QvEofY/gged21hW4T/kvCGaGAHxPxHiNrNMxt4/HvrjdA6T9j/B2FWrORD3P4Vq3xpcS+E/ex1xqsTe+z/hlQuQOrjqP/eAcJjufN4/3rMyXlsm6z+A9V2z9DzsP2/8RXlhbes/k7+o+7h79z8g2wlYJ2PpP3FZfX9zkPI/Yl72HhSG8j8/5LPLlo7pP+KviIZMxOs/uyrGR1I99z9KJhYsl3X1Py7ymNu0ts0/zr+/HMIo3D+wLljMGs/MP2SB+KGRju8/nFCZz7tn6T+fA3C2TPrgP5MkzzsR2Ow/OIlnowZc5T/zG1vO0/TyP9vs5WOz6PA/vymZaKKz+z8DIrRt53jzP8K2i3Mpiuk/EVkaTKU89z+MMBHxlvfwP48DZbuVMu4/VErjkqbu7j9qe9nhgBTwP6oqI+ef/Oc/Br/xTr/07j9cLiOuV5/hP3b2WduOAfY/JJT1fqK6AkCoWu7njVLdP06eUBM6pPc/UsiRHgrM+j+8TY8YlhnRPxJimaDFh/Y/a5oAF5E85T9O9SdpKf3wP8ndUpLe3vo/r4+OBvvA6D/FL/PYEvzeP74d91dghPo/PTkV9tAR5j/yq8dXgAHiP7TQxvlC0uk/oQgqgQYJ7T+UM/MwtrXfP8uWiKQjuus/AFLCvMtG4T9Uun8HxyzkP/22SKYCw94/7O9AWIGA8z+kxycj4P7dP7DioCfrLOE/TCXFZMSd8D8OAA5Lo+XwP1SypYvnle0/wvK+XG3m8D+Wew/nT47xP9LbqfHCT/Q/EjLrLp+t7D9yj4kCSd3jP20iuLaDv+E/0KtoJ8sX7D8jN4tJBw/sP2K4qr355P4/2sYW0wgv8T83PerHdaDmPyT0rHCKCOc/CH9BbpKZ6j/X9kLGftvvP9wYCZDmPPo/UZoZmI0c0T/8VsxLVKnbP2LrUGvUZt4/5/ZTPYAn5T8vBHSZwNfuP6zXg1Btj/M/h+DEXu166z/qw/dWSIvoP5qwambJkuY/u1YygodR0D+XwUPxI0nlP1/LcnryF+E/eau1SY9nzj8+SwFhsU36PwIxKaDRFOc/hNyvLSof4z/dWR+okkjiP7GwEcJIic8/gNA0WJ8N+D+DD7YBSObeP1tStzLKNOE/LGtFMtxp8T8WGaMkXUjdPysxvIjaaeE/f5GnUsZH5z8ev+AOYzzgPzip2prIpuE/EyLempB47T+te7TW2aH1PyEs8rAq3O4/BhEh5+dF5D9I85u6kUD2P1mksfmwKug/6zXsKryE9D9t9/vn8UjnP8QJagppOec/sKFecv1t6T9ctyEmeSHwP6xTu06CzPA/tqcaycRi8z++DKUodsXvPwGsnnvUQtg/bGmgMufT+D+PANZw0xXnP8oUEtY/vMg/i6WNEz3q7j/CS4VyPFXqP1Od19bs5ec/EEaTFqm93z8Ghwo2btjfP2qWDyNdvOI/Ls2GXJq16T/EmBecdsv/P9kYm+4G8vY/BXptI/ut8T9zgGHGIKTvP4z9iE0DmOw/GWX0E6MM4z8hI0xFaWPrP+5ElhKLpvM/RqqK0H658D9NviVkPNbiP6mjI4bpzfQ/IAyRNJWZ4D+n3NND5jjhPynWg5Bly+E/e7tMchP97j8+jxbYHL7xP4guL/BRc+o/yK0f5mmK4D9TNcBL9EPnP3ZcSc2fzfQ/LfkGTYEo3z+FcCmStHLoPyhXXVxO3/8/TV7we8+V5j8D5O3SJXjwPwGOD9i9keQ/yKw9Nehb2D/+HCWSgaTmP1f1nLXoLdo/Duuv/MNF6D8av2Sw4MvrP2/8/jJ8T+I//pr3KfCS5T/YEHZxNVT1P9NlYsf8ses/WDTOucDhAECMAVGoGtH6P6BxGDHirNA/xIEt0Jdr9T9EbtyvZrT4Py1XNFUi998/qmcm80m38T+vQcKW6nbmP8q5ZilpGeE/ryAi6I+B5j+1bWECYEH2P3Cm1rXQ2fI/505TlN7C9D/F/mo84wLoP9e+wnLmI+g/oLoA3JY1AEBpn/6ngmnxP1fe+sHVa+w/Gb4we2lf5j+nn2YrRQfvP+Dwo5vjSfA/u+n+lbEK4D/rnfCtlnrrPx3/ENmfs/E/Fozw1E3i+D9vMTcwz8TyP9T/MER7Nd8/+E9lKNxd8T/EhMRY7LDMP7WKC/Qh6d4/wX4ZyRyZ7j8AGGrxN1r2P5DwYtSkuPU/gAHqq7XY5j/NzgMu0J3QPwuRnK/aPOg/V4D2lZ888D/NEXDrSaXzP4aHa1hwHOg/35j7zpVj3z+872pONj/dPya5sNxiON0/QVdbFKKd+D9bibcTEbPTP1Q2975z/tE/BEay0dJs/D9I3JVoP+LqP+iGPa6aX+U/9S6xrlpK8z8UxW+4O0b+P83/yUZLyug/hZEPNr5N8j8Nfw64pQLxP07SC8sHJ+Y/Jmldmw7z7j/oef0qLsnsP0bDSChCUPM/omOn8L8T5j/MnYYJBMv6P1wYAMh0T/s/67sYCrG55j+C5EMFodfeP9hFd8HH2Os/9Xh9bzZQ/z/5EDiIYa/OP2LD8UeWKOA/vVX9QX5Z9j9BGs8o+k/hP1nQUn4psOQ/UUOA1Ota4D9moQcB/ZPmP5iB94tnp9E//eUkzyfq/D9l1eUCZhbSP8p/t3nwuuc/0zSi0gri+z99Bujjhkf0Pyt7SdkdmNA/MkNMrhBC8T+2vqcIJi3OPw==","dtype":"float64","shape":[287]}},"name":"Series:Banana","selected":{"id":"1361","type":"Selection"},"selection_policy":{"id":"1360","type":"UnionRenderers"}},"id":"1309","type":"ColumnDataSource"},{"attributes":{"fill_color":{"value":"#1f77b4"},"line_color":{"value":"#1f77b4"},"size":{"field":"quantity","units":"screen"},"x":{"field":"date"},"y":{"field":"total_price"}},"id":"1286","type":"Scatter"},{"attributes":{"days":[1,15]},"id":"1301","type":"DaysTicker"},{"attributes":{"fill_alpha":{"value":0.1},"fill_color":{"value":"#1f77b4"},"line_alpha":{"value":0.1},"line_color":{"value":"#1f77b4"},"size":{"field":"quantity","units":"screen"},"x":{"field":"date"},"y":{"field":"total_price"}},"id":"1287","type":"Scatter"},{"attributes":{},"id":"1332","type":"UnionRenderers"},{"attributes":{"callback":null,"renderers":"auto","tooltips":[["Total Price (M $)","@total_price"],["Quantity Sold (M Units)","@quantity"]]},"id":"1393","type":"HoverTool"},{"attributes":{"axis_label_text_color":{"value":"#666666"},"axis_label_text_font_size":{"value":"11pt"},"axis_label_text_font_style":"bold","axis_line_color":{"value":"#C0C0C0"},"formatter":{"id":"1282","type":"NumeralTickFormatter"},"major_label_text_color":{"value":"#898989"},"major_label_text_font_size":{"value":"10pt"},"major_tick_in":0,"major_tick_line_color":{"value":"#C0C0C0"},"major_tick_out":4,"minor_tick_line_color":{"value":"#C0C0C0"},"minor_tick_out":1,"plot":{"id":"1253","subtype":"Figure","type":"Plot"},"ticker":{"id":"1268","type":"BasicTicker"}},"id":"1267","type":"LinearAxis"},{"attributes":{"source":{"id":"1309","type":"ColumnDataSource"}},"id":"1314","type":"CDSView"},{"attributes":{"format":"0,0.[0]"},"id":"1282","type":"NumeralTickFormatter"}],"root_ids":["1253"]},"title":"Bokeh Application","version":"1.0.1"}}';
          var render_items = [{"docid":"afa73682-294e-43f1-b519-fd5c3316f94e","roots":{"1253":"645bb2c2-ed6a-478d-bed6-a9f133d0f676"}}];
          root.Bokeh.embed.embed_items(docs_json, render_items);

          }
          if (root.Bokeh !== undefined) {
            embed_document(root);
          } else {
            var attempts = 0;
            var timer = setInterval(function(root) {
              if (root.Bokeh !== undefined) {
                embed_document(root);
                clearInterval(timer);
              }
              attempts++;
              if (attempts > 100) {
                console.log("Bokeh: ERROR: Unable to run BokehJS code because BokehJS library is missing");
                clearInterval(timer);
              }
            }, 10, root)
          }
        })(window);
      });
    };
    if (document.readyState != "loading") fn();
    else document.addEventListener("DOMContentLoaded", fn);
  })();
</script>
