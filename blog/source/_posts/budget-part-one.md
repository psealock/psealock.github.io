title: Budget Visualization Part One. D3js and Polymer Web Components
date: 2014-11-15 07:29:03
<!-- tags: D3js Polymer Web Components -->
---


### DIY Charting Finances

Upon needing to breakdown my budget, I logged onto [mint.com](https://www.mint.com/) only to discover they don't support New Zealand banks. I want to see where my money is going. So what's a developer to do?

## Having a Play with D3js and Web Components

ANZ supports exporting bank data for the last 30 days in CSV format. Sweet, I want to adapt [mbostock's wonderful pie chart](http://bl.ocks.org/mbostock/3887235) to read CSV, separate purchases into groups and chart it up.

[Demo and full source code](http://bl.ocks.org/psealock/a4f1e24535f0353d91ea)

![](http://i.imgur.com/wll2VVe.jpg)

If I throw everything into a domReady Polymer function, everything works great.

```
<polymer-element name="pie-chart">
  <template>
  </template>
  <script>
    Polymer({
      domReady: function () {
        //... Everything D3 in here
      }
    });
  </script>
</polymer-element>
```

But, in order to take advantage of Polymer's built in data binding, I'll need to go further.

I find most d3 charts fall into 3 steps: set up the elements, draw the chart, and some sort of refresh. I've chosen to create the elements on the Component itself so other funcitons can have access.

```
createElements: function () {
  var width = 500,
      height = 500,
      radius = Math.min(width, height) / 2;

  this.color = d3.scale.ordinal()
      .range(["green", "orange", "blue", "red"]);

  this.arc = d3.svg.arc()
      .outerRadius(radius - 10)
      .innerRadius(0);

  this.pie = d3.layout.pie()
      .sort(null)
      .value(function(d) { return d.amount; });

  this.svg = d3.select(this.$.svg)
      .attr("width", width)
      .attr("height", height)
    .append("g")
      .attr("transform", "translate(" + width / 2 + "," + height / 2 + ")");
},
```
Draw the chart.

```
drawChart: function () {
  var me = this;

  me.data.forEach(function(d) {
    d.amount = +d.amount;
  });

  me.g = me.svg.selectAll(".arc")
      .data(me.pie(me.data))
    .enter().append("g")
      .attr("class", "arc");

  me.g.append("path")
      .attr("d", me.arc)
      .style("fill", function(d) { return me.color(d.data.type); });

  me.g.append("text")
      .attr("transform", function(d) { return "translate(" + me.arc.centroid(d) + ")"; })
      .attr("dy", ".35em")
      .style("text-anchor", "middle")
      .text(function(d) { return d.data.type; });
},
```

To refresh the chart, I'll need to re-bind the data

```
refreshChart: function () {
  var me = this;

  me.g.data(me.pie(me.data))
    .select("path").attr("d", me.arc);

  me.g.select("text")
    .attr("transform", function(d) { return "translate(" + me.arc.centroid(d) + ")"; });
},
```

Lastly, gotta get dat data.

```
getData: function () {
  var me = this;
  d3.csv(me.url, function (error, data) {
    me.data = data;
    me.drawChart();
  });
},
domReady: function () {
  this.createElements();
  this.getData();
},
```

Put it together with some data binding inputs. [Demo and full source code](http://bl.ocks.org/psealock/a4f1e24535f0353d91ea).

### Next Step

Load up some real data and have look.
