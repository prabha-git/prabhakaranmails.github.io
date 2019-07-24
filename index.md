<html>

<head>
    <script src="https://d3js.org/d3.v5.min.js">

        d3.csv("wdi.csv")
            .then(function (data) {
                d3.select("body")
                    .append("text")
                    .text("heloo prabha !")
            }
            );

    </script>

</head>

<body>
    <h1>Narrative Visualization !!!</h1>
</body>


</html>
