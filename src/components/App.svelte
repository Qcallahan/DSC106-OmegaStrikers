<script>
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
    
    let tempData = [];
    let forwardData = [];
    let goalieData = [];
    let currentData = [];
    let selectedCharacters = new Set();
    let isForwards = true; 

    const Strikers = {
            CD_MagicalPlaymaker: '../assets/CloseUp/Ai.Mi.png',
            CD_ShieldUser: '../assets/CloseUp/Asher.png',    
            CD_AngelicSupport: '../assets/CloseUp/Atlas.png',
            CD_NimbleBlaster: '../assets/CloseUp/Drek\'ar.png',
            CD_StalwartProtector: '../assets/CloseUp/Dubu.png',
            CD_EmpoweringEnchanter: '../assets/CloseUp/Era.png',
            CD_TempoSniper: '../assets/CloseUp/Estelle.png',
            CD_GravityMage: '../assets/CloseUp/Finii.png',
            CD_FlexibleBrawler: '../assets/CloseUp/Juliette.png',
            CD_CleverSummoner: '../assets/CloseUp/Juno.png',
            CD_SpeedySkirmisher: '../assets/CloseUp/Kai.png',
            CD_UmbrellaUser: '../assets/CloseUp/Kazan.png',
            CD_ChaoticRocketeer: '../assets/CloseUp/Luna.png',
            CD_DrumOni: '../assets/CloseUp/Mako.png',
            CD_Healer: '../assets/CloseUp/Nao.png',
            CD_EDMOni: '../assets/CloseUp/Octavia.png',
            CD_WhipFighter: '../assets/CloseUp/Rasmus.png',
            CD_ManipulatingMastermind: '../assets/CloseUp/Rune.png',
            CD_RockOni: '../assets/CloseUp/Vyce.png',
            CD_HulkingBeast: '../assets/CloseUp/X.png',
            CD_FlashySwordsman: '../assets/CloseUp/Zentaro.png'
        }

    onMount(async () => {
        const res = await fetch('global.csv'); 
        const csv = await res.text();
        tempData = d3.csvParse(csv, d3.autoType);

        createScatterPlot();
        
        const resForward = await fetch('average_forwards.csv');
        const csvForward = await resForward.text();
        forwardData = d3.csvParse(csvForward, d3.autoType);

        const resGoalie = await fetch('average_goalies.csv');
        const csvGoalie = await resGoalie.text();
        goalieData = d3.csvParse(csvGoalie, d3.autoType);

        currentData = isForwards ? forwardData : goalieData;

        createBarChart();
    });
   function createScatterPlot() {
        const margin = { top: 40, right: 40, bottom: 50, left: 60 };
        const width = 600 - margin.left - margin.right;
        const height = 400 - margin.top - margin.bottom;

        const svg = d3.select('main')
            .append('svg')
            .attr('width', width + margin.left + margin.right)
            .attr('height', height + margin.top + margin.bottom)
            .append('g')
            .attr('transform', `translate(${margin.left},${margin.top})`);

        const yScale = d3.scaleLinear()
            .domain([-100, 4500])
            .range([height, 0]);

        const xScale = d3.scaleLinear()
            .domain([-100, 4500])
            .range([0, width]);

        // Define color scale with 21 unique colors
        const colorScale = d3.scaleOrdinal()
            .domain(tempData.map(d => d.most_played_character))
            .range(["#1f77b4", "#aec7e8", "#ff7f0e", "#ffbb78", "#2ca02c", "#98df8a", "#d62728", "#ff9896", "#9467bd", "#c5b0d5", "#8c564b", "#c49c94", "#e377c2", "#f7b6d2", "#7f7f7f", "#c7c7c7", "#bcbd22", "#dbdb8d", "#17becf", "#9edae5", "#1a55FF"]); // Add your 21 colors here

        svg.append('g')
            .attr('transform', `translate(0,${height})`)
            .call(d3.axisBottom(xScale))
            .append('text')
            .attr('x', width / 2)
            .attr('y', margin.bottom - 10)
            .attr('fill', '#000')
            .attr('text-anchor', 'middle')
            .text('Losses');

        svg.append('g')
            .call(d3.axisLeft(yScale))
            .append('text')
            .attr('transform', 'rotate(-90)')
            .attr('x', -height / 2)
            .attr('y', -margin.left + 20)
            .attr('fill', '#000')
            .attr('text-anchor', 'middle')
            .text('Wins');

        const circles = svg.selectAll('circle')
            .data(tempData)
            .enter()
            .append('circle')
            .attr('cx', d => xScale(d.losses))
            .attr('cy', d => yScale(d.wins))
            .attr('r', 5)
            .style('fill', d => colorScale(d.most_played_character))
            .style('opacity', d => selectedCharacters.has(d.most_played_character) ? .8 : 0.05)
            .on('mouseover', handleMouseOver)
            .on('mouseout', handleMouseOut);


        // Create buttons for each character option
        const characterOptions = Array.from(new Set(tempData.map(d => d.most_played_character)));
        const buttons = d3.select('main')
            .insert('div', 'svg')
            .attr('class', 'buttons-container')
            .selectAll('button')
            .data(characterOptions)
            .enter()
            .append('button')
            .text(d => d)
            .on('click', toggleCharacter)
            .style('background-color', d => selectedCharacters.has(d) ? '#66bb6a' : '');


        // Select All button
        d3.select('main')
            .insert('button', 'svg')
            .text('Select All')
            .on('click', selectAll)
            .style('background-color', '');

        // Deselect All button
        d3.select('main')
            .insert('button', 'svg')
            .text('Deselect All')
            .on('click', deselectAll)
            .style('background-color', '');


        function toggleCharacter(event) {
            const character = event.target.textContent;
            if (selectedCharacters.has(character)) {
                selectedCharacters.delete(character);
                d3.select(this).style('background-color', '');
            } else {
                selectedCharacters.add(character);
                d3.select(this).style('background-color', '#66bb6a');
            }

            svg.selectAll('circle')
                .style('opacity', d => selectedCharacters.has(d.most_played_character) ? .8 : 0.05);

            circles.sort((a, b) => {
                if (selectedCharacters.size === 0) return 0; // No selected characters, maintain order
                const aSelected = selectedCharacters.has(a.most_played_character);
                const bSelected = selectedCharacters.has(b.most_played_character);
                if (aSelected && !bSelected) return 1; // A is selected character and B is not, move A to front
                if (!aSelected && bSelected) return -1; // B is selected character and A is not, move B to front
                if (aSelected && bSelected) {
                    // Both A and B are selected characters, prioritize the most recently selected character
                    if (a.most_played_character === character) return 1; // A is the most recently selected character, move to front
                    if (b.most_played_character === character) return -1; // B is the most recently selected character, move to front
                }
                return 0; // Other cases, maintain order
            });
            
            createBarChart(selectedCharacters);
        }

        function selectAll() {
            characterOptions.forEach(character => selectedCharacters.add(character));

            svg.selectAll('circle')
                .style('opacity', .8);

            buttons.style('background-color', '#66bb6a');
            createBarChart(selectedCharacters);
        }

        function deselectAll() {
            selectedCharacters.clear();

            svg.selectAll('circle')
                .style('opacity', 0.05);

            buttons.style('background-color', '');
            createBarChart(selectedCharacters);
        }

        const tooltip = d3.select('body').append('div')
            .attr('class', 'tooltip')
            .style('opacity', 0);

        function handleMouseOver(event, d) {
            const topRole = d.topRole;
            const rank = d.rank;
            const mostPlayedCharacter = d.most_played_character;
            const username = d.username;

            d3.select(this)
                .style('stroke', 'black')
                .style('stroke-width', 2);

            const tooltip = d3.select('main').append('div')
                .attr('class', 'tooltip')
                .style('opacity', 0)
                .html(`<strong>Username:</strong> ${username}<br><strong>Top Role:</strong> ${topRole}<br><strong>Rank:</strong> ${rank}<br><strong>Most Played Character:</strong> ${mostPlayedCharacter}`);

            const tooltipWidth = parseInt(tooltip.style('width'));
            const tooltipHeight = parseInt(tooltip.style('height'));
            console.log(event.pageX);
            tooltip.style('left', (event.pageX - tooltipWidth / 2) + 'px')
                .style('top', (event.pageY - tooltipHeight - 10) + 'px')
                .transition()
                .duration(200)
                .style('opacity', 1);
        }

        function handleMouseOut() {
            d3.select(this)
                .style('stroke', 'none');

            d3.select('.tooltip')
                .style('opacity', 0)
                .remove();
        }
    }

    function createBarChart(selectedCharacters) {
        const margin = { top: 40, right: 40, bottom: 70, left: 60 };
        const width = 600 - margin.left - margin.right;
        const height = 450 - margin.top - margin.bottom;

        // Remove existing chart
        d3.select('#bar-chart').selectAll('*').remove();

        const svg = d3.select('#bar-chart')
            .attr('width', width + margin.left + margin.right)
            .attr('height', height + margin.top + margin.bottom)
            .append('g')
            .attr('transform', `translate(${margin.left},${margin.top})`);

        let filteredData = currentData.filter(d => selectedCharacters.has(d.striker) || d.striker === "Global Average");

        // Sort the data based on average saves
        filteredData.sort((a, b) => b.average_saves - a.average_saves);

        const xScale = d3.scaleBand()
            .domain(filteredData.map(d => d.striker))
            .range([0, width])
            .padding(0.1);

        const yScale = d3.scaleLinear()
            .domain([0, d3.max(filteredData, d => d.average_saves)])
            .nice()
            .range([height, 0]);

        svg.selectAll('rect')
            .data(filteredData)
            .enter()
            .append('rect')
            .attr('x', d => xScale(d.striker))
            .attr('y', d => yScale(d.average_saves))
            .attr('width', xScale.bandwidth())
            .attr('height', d => height - yScale(d.average_saves))
            .attr('fill', d => d.striker === "Global Average" ? "red" : "#69b3a2")
            .on('mouseenter', function (event, d) {
                const imageName = Strikers[d.striker]; 
                showImageAtCursor(event, imageName);
            })
            .on('mouseleave', function () {
                // Delay hiding the image slightly
                if (isImageVisible) {
                    clearTimeout(imageTimeout);
                    imageTimeout = setTimeout(function () {
                        hideImage();
                        isImageVisible = false;
                    }, 100);
                }
            });


        svg.append('g')
            .attr('transform', `translate(0,${height})`)
            .call(d3.axisBottom(xScale))
            .selectAll('text')
            .style('text-anchor', 'end')
            .attr('transform', 'rotate(-45)');

        svg.append('g')
            .call(d3.axisLeft(yScale));
    }


    function switchData(event) {
        isForwards = event.target.value === "forwards";
        currentData = isForwards ? forwardData : goalieData;
        createBarChart(selectedCharacters);
    }

    // Function to show the image at an offset from the cursor position
    function showImageAtCursor(event, imageUrl) {
        console.log('Image URL:', imageUrl);
        const img = document.createElement('img');
        img.src = imageUrl;
        img.style.position = 'absolute';
        
        // Adjust the offset as needed
        const offset = 10;
        img.style.top = `${event.clientY + offset}px`;
        img.style.left = `${event.clientX + offset}px`;
        
        img.style.maxWidth = '100px'; // Adjust as needed
        img.id = 'character-image';
        
        document.body.appendChild(img);

        function updateImagePosition(event) {
            // Update the image position with the offset
            img.style.top = `${event.clientY + offset}px`;
            img.style.left = `${event.clientX + offset}px`;
        }

        function hideImage() {
            document.removeEventListener('mousemove', updateImagePosition);
            img.parentNode.removeChild(img); // Remove the image when the mouse leaves it
        }

        document.addEventListener('mousemove', updateImagePosition);

        // Remove the event listener and hide the image when mouse leaves the chart area
        svg.on('mouseleave', hideImage);
    };

    function hideImage() {
        console.log("aaaa");
        const img = document.getElementById('character-image');
        if (img) {
            img.parentNode.removeChild(img);
        }
    }

</script>

<main>
    <svg id="scatter-plot"></svg>
    <select on:change={switchData}>
        <option value="forwards">Forwards</option>
        <option value="goalies">Goalies</option>
    </select>

    <svg id="bar-chart"></svg>
</main>


<style>
    .tooltip {
        position: absolute;
        text-align: center;
        width: 120px;
        height: auto;
        padding: 6px;
        font: 12px sans-serif;
        background: #ddd;
        border: 0px;
        border-radius: 8px;
        pointer-events: none;
    }
</style>
