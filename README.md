# luau1251.github.io
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tabla Interactiva de Proveedores de Servicios con Gráficas</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://d3js.org/d3.v7.min.js"></script>
    <style>
        /* Custom styles for better aesthetics */
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f0f4f8; /* Light blue-gray background */
            display: flex;
            justify-content: center;
            align-items: flex-start; /* Align to top to prevent table from being too high */
            min-height: 100vh; /* Ensure it takes full viewport height */
            padding: 2rem; /* Add some padding around the content */
            box-sizing: border-box; /* Include padding in element's total width and height */
        }
        .container {
            background-color: #ffffff;
            padding: 2.5rem;
            border-radius: 1.5rem; /* More rounded corners */
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1); /* Softer shadow */
            max-width: 1200px; /* Max width for larger screens */
            width: 100%;
        }
        table {
            width: 100%;
            border-collapse: separate; /* Allows for rounded corners on cells */
            border-spacing: 0;
            margin-top: 1.5rem;
        }
        th, td {
            padding: 1rem 1.25rem; /* Increased padding */
            text-align: left;
            border-bottom: 1px solid #e2e8f0; /* Light gray border */
        }
        th {
            background-color: #4a90e2; /* Blue header */
            color: white;
            font-weight: 600; /* Slightly bolder */
            position: sticky; /* Sticky header */
            top: 0;
            z-index: 10; /* Ensure header stays on top */
            cursor: pointer; /* Indicate sortable columns */
            transition: background-color 0.2s ease-in-out;
        }
        th:hover {
            background-color: #3a7bd5; /* Darker blue on hover */
        }
        tr:last-child td {
            border-bottom: none; /* No border on the last row */
        }
        tbody tr:hover {
            background-color: #f7fafc; /* Lighter hover effect */
        }
        input[type="text"] {
            border: 1px solid #cbd5e0; /* Light gray border */
            padding: 0.75rem 1rem;
            border-radius: 0.75rem; /* Rounded input */
            flex-grow: 1; /* Allow input to grow */
            transition: border-color 0.2s ease-in-out, box-shadow 0.2s ease-in-out;
        }
        input[type="text"]:focus {
            outline: none;
            border-color: #4a90e2; /* Blue focus border */
            box-shadow: 0 0 0 3px rgba(74, 144, 226, 0.2); /* Soft focus shadow */
        }
        .search-container {
            margin-bottom: 1.5rem;
            display: flex; /* Use flexbox for alignment */
            gap: 1rem; /* Space between input and button */
            align-items: center;
        }
        .table-scroll {
            overflow-x: auto; /* Enable horizontal scrolling for small screens */
            border-radius: 1rem; /* Rounded corners for the scrollable area */
            border: 1px solid #e2e8f0; /* Border around the table */
        }
        .clear-button {
            background-color: #ef4444; /* Red for clear button */
            color: white;
            padding: 0.75rem 1.25rem;
            border-radius: 0.75rem;
            border: none;
            cursor: pointer;
            transition: background-color 0.2s ease-in-out, box-shadow 0.2s ease-in-out;
        }
        .clear-button:hover {
            background-color: #dc2626; /* Darker red on hover */
            box-shadow: 0 4px 10px rgba(239, 68, 68, 0.2);
        }
        /* Style for sort indicators */
        .sort-indicator {
            margin-left: 0.5rem;
            font-size: 0.8em;
        }
        /* Styles for the D3 chart */
        .chart-container {
            margin-bottom: 2.5rem;
            padding: 1.5rem;
            background-color: #f7fafc;
            border-radius: 1rem;
            box-shadow: inset 0 2px 5px rgba(0, 0, 0, 0.05);
            text-align: center;
        }
        .chart-title {
            font-size: 1.5rem;
            font-weight: bold;
            color: #333;
            margin-bottom: 1.5rem;
        }
        .bar {
            fill: #4a90e2; /* Blue color for bars */
            transition: fill 0.3s ease;
        }
        .bar:hover {
            fill: #3a7bd5; /* Darker blue on hover */
        }
        .axis text {
            font-size: 0.85rem;
            fill: #555;
        }
        .axis path,
        .axis line {
            stroke: #ccc;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1 class="text-3xl font-bold text-gray-800 mb-6 text-center">Tabla de Proveedores de Servicios en la Nube</h1>

        <div class="chart-container">
            <div class="chart-title">Espacio Gratuito Ofrecido (GB)</div>
            <svg id="freeSpaceChart" class="w-full h-64"></svg>
        </div>

        <div class="search-container">
            <input type="text" id="searchInput" onkeyup="filterTable()" placeholder="Buscar en la tabla..." class="text-base">
            <button class="clear-button" onclick="clearSearch()">Limpiar Búsqueda</button>
        </div>

        <div class="table-scroll">
            <table id="serviceTable">
                <thead>
                    <tr>
                        <th onclick="sortTable(0)">Proveedor <span class="sort-indicator"></span></th>
                        <th onclick="sortTable(1)">Servicios (Personal) <span class="sort-indicator"></span></th>
                        <th onclick="sortTable(2)">Servicios (Empresarial) <span class="sort-indicator"></span></th>
                        <th onclick="sortTable(3)">Ventajas <span class="sort-indicator"></span></th>
                        <th onclick="sortTable(4)">Desventajas <span class="sort-indicator"></span></th>
                    </tr>
                </thead>
                <tbody>
                    </tbody>
            </table>
        </div>
    </div>

    <script>
        // Data for the table and chart
        const serviceData = [
            {
                provider: "Google Drive",
                personalServices: "15 GB gratis, integración con Gmail",
                businessServices: "Google Workspace: Drive, Docs, Sheets, Meet, etc.",
                advantages: "Fácil de usar, integración con otros servicios de Google",
                disadvantages: "Espacio limitado en plan gratuito",
                freeSpaceGB: 15 // Added numerical value for charting
            },
            {
                provider: "Dropbox",
                personalServices: "2 GB gratis, sincronización automática",
                businessServices: "Dropbox Business: administración de equipos, seguridad",
                advantages: "Muy buena sincronización y restauración de versiones",
                disadvantages: "Espacio gratuito muy limitado",
                freeSpaceGB: 2 // Added numerical value for charting
            },
            {
                provider: "Amazon S3",
                personalServices: "No está pensado para uso personal",
                businessServices: "Escalabilidad masiva, backups, hosting, IA y Big Data",
                advantages: "Alta seguridad y personalización, ideal para empresas",
                disadvantages: "Complejidad técnica, no es amigable para usuarios básicos",
                freeSpaceGB: 0 // No personal free space for charting
            }
        ];

        let currentSortColumn = -1; // -1 means no column is sorted
        let sortDirection = 'asc'; // 'asc' for ascending, 'desc' for descending

        /**
         * Populates the table with data from the serviceData array.
         * Clears existing rows before adding new ones.
         */
        function populateTable() {
            const tableBody = document.querySelector("#serviceTable tbody");
            tableBody.innerHTML = ''; // Clear existing rows

            serviceData.forEach(rowData => {
                const row = document.createElement("tr");
                row.innerHTML = `
                    <td class="whitespace-normal">${rowData.provider}</td>
                    <td class="whitespace-normal">${rowData.personalServices}</td>
                    <td class="whitespace-normal">${rowData.businessServices}</td>
                    <td class="whitespace-normal">${rowData.advantages}</td>
                    <td class="whitespace-normal">${rowData.disadvantages}</td>
                `;
                tableBody.appendChild(row);
            });
        }

        /**
         * Filters the table rows based on the search input value.
         * Rows that do not contain the search text (case-insensitive) are hidden.
         */
        function filterTable() {
            const input = document.getElementById("searchInput");
            const filter = input.value.toLowerCase();
            const table = document.getElementById("serviceTable");
            const tr = table.getElementsByTagName("tr");

            // Loop through all table rows, and hide those that don't match the search query
            for (let i = 1; i < tr.length; i++) { // Start from 1 to skip the header row
                let rowMatches = false;
                const td = tr[i].getElementsByTagName("td");
                for (let j = 0; j < td.length; j++) {
                    const cellText = td[j].textContent || td[j].innerText;
                    if (cellText.toLowerCase().includes(filter)) { // Use .includes() for partial matching
                        rowMatches = true;
                        break;
                    }
                }
                if (rowMatches) {
                    tr[i].style.display = "";
                } else {
                    tr[i].style.display = "none";
                }
            }
        }

        /**
         * Clears the search input field and shows all table rows.
         */
        function clearSearch() {
            document.getElementById("searchInput").value = '';
            filterTable(); // Re-filter to show all rows
        }

        /**
         * Sorts the table by the specified column index.
         * Toggles between ascending and descending order on successive clicks of the same column.
         * @param {number} columnIndex - The index of the column to sort by.
         */
        function sortTable(columnIndex) {
            const table = document.getElementById("serviceTable");
            const tbody = table.querySelector("tbody");
            const rows = Array.from(tbody.querySelectorAll("tr")); // Get all rows as an array

            // Determine sort direction
            if (currentSortColumn === columnIndex) {
                sortDirection = sortDirection === 'asc' ? 'desc' : 'asc';
            } else {
                currentSortColumn = columnIndex;
                sortDirection = 'asc'; // Default to ascending for a new column
            }

            // Sort the rows
            rows.sort((a, b) => {
                const aText = a.children[columnIndex].textContent.toLowerCase();
                const bText = b.children[columnIndex].textContent.toLowerCase();

                if (aText < bText) {
                    return sortDirection === 'asc' ? -1 : 1;
                }
                if (aText > bText) {
                    return sortDirection === 'asc' ? 1 : -1;
                }
                return 0;
            });

            // Re-append sorted rows to the tbody
            rows.forEach(row => tbody.appendChild(row));

            // Update sort indicators in headers
            updateSortIndicators();
        }

        /**
         * Updates the sort indicators (arrows) in the table headers
         * to reflect the current sort order.
         */
        function updateSortIndicators() {
            const headers = document.querySelectorAll("#serviceTable th");
            headers.forEach((header, index) => {
                const indicator = header.querySelector(".sort-indicator");
                if (indicator) {
                    indicator.textContent = ''; // Clear all indicators first
                    if (index === currentSortColumn) {
                        indicator.textContent = sortDirection === 'asc' ? '▲' : '▼';
                    }
                }
            });
        }

        /**
         * Draws the bar chart using D3.js.
         * The chart visualizes the 'freeSpaceGB' for each provider.
         */
        function drawChart() {
            // Clear any existing SVG content
            d3.select("#freeSpaceChart").selectAll("*").remove();

            const svg = d3.select("#freeSpaceChart");
            const containerWidth = svg.node().getBoundingClientRect().width;
            const containerHeight = svg.node().getBoundingClientRect().height;

            const margin = { top: 20, right: 30, bottom: 60, left: 50 }; // Increased bottom margin for labels
            const width = containerWidth - margin.left - margin.right;
            const height = containerHeight - margin.top - margin.bottom;

            const g = svg.append("g")
                .attr("transform", `translate(${margin.left},${margin.top})`);

            // X scale for providers (ordinal scale)
            const x = d3.scaleBand()
                .rangeRound([0, width])
                .padding(0.1)
                .domain(serviceData.map(d => d.provider));

            // Y scale for free space (linear scale)
            const y = d3.scaleLinear()
                .rangeRound([height, 0])
                .domain([0, d3.max(serviceData, d => d.freeSpaceGB) + 5]); // Add some padding to the max Y value

            // X axis
            g.append("g")
                .attr("class", "axis axis--x")
                .attr("transform", `translate(0,${height})`)
                .call(d3.axisBottom(x))
                .selectAll("text") // Select all text elements in the x-axis
                .attr("transform", "rotate(-45)") // Rotate labels for better readability
                .style("text-anchor", "end"); // Anchor text to the end of the rotation

            // Y axis
            g.append("g")
                .attr("class", "axis axis--y")
                .call(d3.axisLeft(y))
                .append("text")
                .attr("fill", "#000")
                .attr("transform", "rotate(-90)")
                .attr("y", 6)
                .attr("dy", "0.71em")
                .attr("text-anchor", "end")
                .text("Espacio Gratuito (GB)");

            // Bars
            g.selectAll(".bar")
                .data(serviceData)
                .enter().append("rect")
                .attr("class", "bar")
                .attr("x", d => x(d.provider))
                .attr("y", d => y(d.freeSpaceGB))
                .attr("width", x.bandwidth())
                .attr("height", d => height - y(d.freeSpaceGB));
        }

        // Populate the table and draw the chart when the page loads
        document.addEventListener("DOMContentLoaded", () => {
            populateTable();
            updateSortIndicators(); // Initialize indicators
            drawChart(); // Draw the chart
        });

        // Redraw chart on window resize for responsiveness
        window.addEventListener('resize', drawChart);
    </script>
</body>
</html>
