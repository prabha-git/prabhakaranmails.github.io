<html>

<head>
    <script src="https://d3js.org/d3.v5.min.js"></script>
    <style>
        * {
            box-sizing: border-box;
        }

        .chart {
            width: 50%;
            float: left;
            padding: 15px;
            border-right: 1px solid white;
        }

        .info {
            width: 50%;
            float: left;
            padding: 15px;
            border: 1px solid red;
        }

        body {
            font: 15px sans-serif;
            background-color: black;
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
            width: 100px;
            height: 60px;
            padding: 2px;
            font: 12px sans-serif;
            background: white;
            border: 0px;
            border-radius: 8px;
            color: black;
            pointer-events: none;
            box-sizing: border-box;
        }


        .tick line {
            visibility: hidden
        }

        #country_text_window {
            color: white;
        }

        .tick text {
            stroke: white;
            font-size: 14px;
        }

        .axis .domain {
            stroke: white;
        }

        h1, p {
            color: white;
        }


    </style>
</head>

<body onload="myFunction()">
    <h1>
        <center>CO<sub>2</sub> emission by G20 Countires</center>
    </h1>
    <script>

        country_map = {
            "China": "http://ontheworldmap.com/china/china-location-map-max.jpg",
            "Japan": "http://ontheworldmap.com/japan/japan-location-map-max.jpg",
            "United States": "http://ontheworldmap.com/usa/usa-location-map-max.jpg",
            "Germany": "http://ontheworldmap.com/germany/germany-location-map-max.jpg",
            "India": "http://ontheworldmap.com/india/india-location-map-max.jpg",
            "Canada": "http://ontheworldmap.com/canada/canada-location-map-max.jpg",
            "United Kingdom": "http://ontheworldmap.com/uk/uk-location-map-max.jpg",
            "Russian Federation": "http://ontheworldmap.com/russia/russia-location-map-max.jpg",
            "Brazil": "http://ontheworldmap.com/brazil/brazil-location-map-max.jpg",
            "Saudi Arabia": "http://ontheworldmap.com/saudi-arabia/saudi-arabia-location-map-max.jpg"
        }

        narrative={
            "1995": "Greenland ice cores suggest that great climate changes (at least on a regional scale) can occur in the space of a single decade. IPCC report detects 'signature' of human-caused greenhouse effect warming, declares that serious warming is likely in the coming century",
            "2000": "A 'Super El Niño' makes this an exceptionally warm year. Toyota introduces Prius, first mass-market electric hybrid car; swift progress in large wind turbines, solar electricity, and other energy alternatives. Global Climate Coalition dissolves as many corporations grapple with threat of warming.",
            "2005" : "Third IPCC report states baldly that global warming, unprecedented since the end of the last ice age, is 'very likely' with highly damaging future impacts. Kyoto treaty goes into effect, signed by major industrial nations except US.",
            "2010" : "Fourth IPCC report warns that serious effects of warming have become evident; cost of reducing emissions would be far less than the damage they will cause. Copenhagen conference fails to negotiate binding agreements: end of hopes of avoiding dangerous future climate change.",
            "2014" : "Researchers find collapse of West Antarctic ice sheet is irreversible, will bring meters of sea-level rise over future centuries. Paris Agreement: nearly all nations pledge to set targets for their own greenhouse gas cuts and to report their progress."
        };

        var format_co2=d3.format(",.2s")

        // Dimensions of the Chart
        var margin = { top: 20, right: 20, bottom: 30, left: 60 },
            width = 960 - margin.left - margin.right,
            height = 600 - margin.top - margin.bottom;

        // setup x axis scale
        var xscale = d3.scaleLinear().range([0, width]);

        // setup y axis scale
        var yscale = d3.scaleLog().range([height, 0])


        function render(year) {


            d3.select("#narrative_text")
                .text(year +" : "+narrative[year]);


            // Loading the data from Github , if the data load is successful we need to proceed with building the graph.
            d3.csv("https://prabhakaranmails.github.io/wdi_g20_1990.csv").then(function (data) {

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
                //yscale.domain([d3.min(data, function (d) { return d.CO2; }), d3.max(data, function (d) { return d.CO2; })]).nice();
                yscale.domain([d3.min(data, function (d) { return d.CO2; }), 10000000]).nice();


                // Creating an SVG element
                var svg = d3.select("#chart_svg")
                    .attr("viewBox", "0 0 " + (width + margin.left + margin.right) + " " + (height + margin.top + margin.bottom+50))
                    //.attr("height", height + margin.top + margin.bottom + 20)
                    //.attr("heigh",100)
                    .append("g")
                    .attr("transform", "translate(" + margin.left + "," + margin.top + ")");



                // Adding scales to axis & adding the axis to svg element.
                var x_axis = d3.axisBottom().scale(xscale).tickFormat(d3.format(".2s"));
                svg.append("g")
                    .attr("transform", "translate(0," + height + ")")
                    .attr("class", "axis")
                    .call(x_axis);

                // Adding a x-axis label
                svg.append("text")
                    .attr("transform", "translate(" + (width / 2) + "," + (height + margin.top + 20) + ")")
                    .style("text-anchor", "middle")
                    .text("GDP")
                    .attr("fill", "white")
                    .style("font-size", 24)


                // Adding scales to axis & adding the axis to svg element.
                var y_axis = d3.axisLeft().scale(yscale).tickFormat(d3.format(".0s"));
                svg.append("g")
                    .attr("class", "axis")
                    .call(y_axis)
                    .selectAll(".tick text")
                    .text(null)
                    .filter(powerOfTen)
                    .text(10)
                    .append("tspan")
                    .attr("dy", "-.7em")
                    .text(function (d) { return Math.round(Math.log(d) / Math.LN10); });


                svg.append("text")
                    .attr("x", -height / 2)
                    .attr("y", -40)
                    .style("text-anchor", "middle")
                    .text("CO2 in Kt (log scale)")
                    .attr("transform", "rotate(-90)")
                    .attr("fill", "white")
                    .style("font-size", 24)



                // Adding tooltip container 
                var tooltip = d3.select("body").append("div")
                    .attr("class", "tooltip")
                    .style("opacity", 0);


                // Tooltip mouseover handler
                var tipMouseover = function (d) {

                    d3.select(this).transition()
                        .duration(100)
                        .ease(d3.easeBounce)
                        .attr("r", 32);

                    var html = d.Country + "<br/>" + "CO2 : "+format_co2(d.CO2)  + "<br/>" + "GDP : "+format_co2(d.GDP) 
                    tooltip.transition()
                        .duration(10)
                        .style("opacity", .9)
                        .style("left", (d3.event.pageX + 15) + "px")
                        .style("top", (d3.event.pageY - 28) + "px");
                    tooltip.html(html);

                };


                // tooltip mouseout event handler
                var tipMouseout = function (d) {

                    d3.select(this).transition()
                        .duration(100)
                        .ease(d3.easeBounce)
                        .attr("r", 14);

                    tooltip.transition()
                        .duration(300)
                        .style("opacity", 0);
                };



                // Adding the elemets to the scatter plot

                svg.selectAll("circle")
                    .data(data)
                    .enter()
                    .append("circle")
                    .filter(function (d) { return (d.Time == year && d.GDP > 0 && d.Pop > 0 && d.CO2 > 0); })
                    .attr("cx", function (d) { return xscale(d.GDP); })
                    .attr("cy", function (d) { return yscale(d.CO2); })
                    .attr("r", 14)
                    .style("fill", "steelblue")
                    .style("opacity", .8)
                    .style("stroke","white")
                    .on("mouseover", tipMouseover)
                    .on("mouseout", tipMouseout)
                    .on("click", function (d) {

                        d3.select("#img_svg").select("image").attr("xlink:href", country_map[d.Country])

                        d3.select("#country_text_window").selectAll("*").remove();
                        d3.select("#country_text_window").append("p").text(" Country : " + d.Country).attr("fill", "white");
                        d3.select("#country_text_window").append("p").text(" Year : " + d.Time).attr("fill", "white");
                        d3.select("#country_text_window").append("p").text(" GDP : " + d.GDP)
                        d3.select("#country_text_window").append("p").text(" Population : " + d.Pop)


                    })
                    .filter(function (d) { return (d.Time == year && d.GDP > 0 && d.CO2 > 500000); })
                    .style("fill", "red")
                    .style("stroke","white")


                // Adding text to only few countries based on the below threshold specified
                svg.selectAll("text")
                    .data(data)
                    .enter()
                    .append("text")
                    .filter(function (d) { return (d.Time == year && d.GDP > 0 && d.CO2 > 500000); })
                    .attr("x", function (d) { return xscale(d.GDP); })
                    .attr("y", function (d) { return yscale(d.CO2); })
                    .attr("font-size", "12px")
                    .style("fill", "white")
                    .text(function (d) { return d.Country; });

            }
            )

        }


        function select(year) {
            d3.select("#chart_svg").selectAll("*").remove();
            render(year)
        }

        function powerOfTen(d) {
            return d / Math.pow(10, Math.ceil(Math.log(d) / Math.LN10 - 1e-12)) === 1;
        }

        function myFunction() {

            var img = d3.select("#img_svg").attr("width", 350).attr("height", 200)


            img.append("g")
                .append("svg:image").attr("xlink:href", "http://ontheworldmap.com/usa/usa-location-map-max.jpg")
                .attr("width", 350)
                .attr("heigh", 300)


        }

    </script>


    <div class="myButton">
        <button onclick="select('1995')">
            1995
        </button>
        <button onclick="select('2000')">
            2000
        </button>
        <button onclick="select('2005')">
            2005
        </button>
        <button onclick="select('2010')">
            2010
        </button>
        <button onclick="select('2014')">
            2014
        </button>
        <button onclick="select()">
            Clear
        </button>
    </div>

 
    <div class ="chart" id="chart_div">
    <p id ="narrative_text"></p>
    <svg id="chart_svg"> </svg>
    </div>
 
    <div id="info_window" class="info">
        <center> <svg id="img_svg"></svg></center>
    </div>

    <div id="country_text_window">
        <p></p>
    </div>

  


</body>


</html>
