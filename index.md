<html>

<head>
    <script src="https://d3js.org/d3.v5.min.js"></script>
    <style>
        body {
            font: 15px sans-serif;
        }

        .axis path,
        .axis line {
            fill: none;
            stroke: #000;
            shape-rendering: crispEdges;
        }

        .dot {
            stroke: none;
        }

        .tooltip {
            position: absolute;
            text-align: center;
            width: 60px;
            height: 28px;
            padding: 2px;
            font: 12px sans-serif;
            background: lightsteelblue;
            border: 0px;
            border-radius: 8px;
            pointer-events: none;
        }
    </style>
</head>

<body>

    <script>

        // Dimensions of the Chart
        var margin = { top: 20, right: 20, bottom: 30, left: 0 },
            width = 960 - margin.left - margin.right,
            height = 500 - margin.top - margin.bottom;

        // setup x axis scale
        var xscale = d3.scaleLinear().range([0, width]);

        // setup y axis scale
        var yscale = d3.scaleLinear().range([height, 0])

        // Loading the data from Github , if the data load is successful we need to proceed with building the graph.
        d3.csv("https://prabhakaranmails.github.io/wdi_g20_1990.csv").then( function (data) {

                // converting the text to numeric for the below variables
                data.forEach(function (d) {
                    d.Time = parseInt(d.Time);
                    d.GDP = parseInt(d.GDP);
                    d.HCI = parseInt(d.HCI);
                    d.Pop = parseInt(d.Pop);
                    d.CO2 = parseInt(d.CO2);
                })


                // based on the data , define the domain of x-axis & y-axis
                xscale.domain([d3.min(data, function (d) { return d.GDP; }), d3.max(data, function (d) { return d.GDP; })]).nice();
                yscale.domain([d3.min(data, function (d) { return d.CO2; }), d3.max(data, function (d) { return d.CO2; })]).nice();


                // Creating an SVG element
                var svg = d3.select("svg")
                    .attr("width", width + margin.left + margin.right)
                    .attr("height", height + margin.top + margin.bottom)
                    .append("g")
                    .attr("transform", "translate(" + margin.left + "," + margin.top + ")");



                // Adding scales to axis & adding the axis to svg element.
                var x_axis = d3.axisBottom().scale(xscale).tickFormat(d3.format(".2s"));
                svg.append("g")
                    .attr("transform", "translate(0," + height + ")")
                    .call(x_axis);

                // Adding a x-axis label
                svg.append("text")
                    .attr("transform", "translate(" + (width / 2) + "," + (height + margin.top + 10) + ")")
                    .style("text-anchor", "middle")
                    .text("GDP")


                // Adding scales to axis & adding the axis to svg element.
                var y_axis = d3.axisLeft().scale(yscale).tickFormat(d3.format(".0s"));
                svg.append("g")
                    .call(y_axis)

                svg.append("text")
                    .attr("x", -height / 2)
                    .attr("y", -30)
                    .style("text-anchor", "middle")
                    .text("CO2")
                    .attr("transform", "rotate(-90)")



                // Adding tooltip container 
                var tooltip = d3.select("body").append("div")
                    .attr("class", "tooltip")
                    .style("opacity", 0);


                // Tooltip mouseover handler
                var tipMouseover = function (d) {
                    console.log(d.Country)
                    var html = d.Country + "<br/>" + d.Pop;
                    tooltip.transition()
                        .duration(200)
                        .style("opacity", .9)
                        .style("left", (d3.event.pageX + 15) + "px")
                        .style("top", (d3.event.pageY - 28) + "px");
                    tooltip.html(html);

                };


                // tooltip mouseout event handler
                var tipMouseout = function (d) {
                    tooltip.transition()
                        .duration(300)
                        .style("opacity", 0);
                };



                // Adding the elemets to the scatter plot

                svg.selectAll("circle")
                    .data(data)
                    .enter()
                    .append("circle")
                    .filter(function (d) { return (d.Time == 2014 && d.GDP > 0 && d.Pop > 0 && d.CO2 > 0); })
                    .attr("cx", function (d) { return xscale(d.GDP); })
                    .attr("cy", function (d) { return yscale(d.CO2); })
                    .attr("r", 10)
                    .style("fill", "steelblue")
                    .style("opacity", 0.8)
                    .on("mouseover", tipMouseover)
                    .on("mouseout", tipMouseout);


                // Adding text to only few countries based on the below threshold specified
                svg.selectAll("text")
                    .data(data)
                    .enter()
                    .append("text")
                    .filter(function (d) { return (d.Time == 2014 && d.GDP > 0 && d.CO2 > 1000000); })
                    .attr("x", function (d) { return xscale(d.GDP); })
                    .attr("y", function (d) { return yscale(d.CO2); })
                    .attr("font-size", "12px")
                    .text(function (d) { return d.Country; });









            }
        )
    </script>

    <svg></svg>

</body>


</html>
