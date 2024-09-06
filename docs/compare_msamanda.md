---
theme: dashboard
title: comparison MSAmanda results
toc: false
---

# Comparison MSAmanda pipelines 

<!-- Load and transform the data -->
<script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
<script>
function stringToNumericList(listAsString) {
  return listAsString
    .slice(1, -1)
    .split(', ')
    .map(Number);
}
// Convert a string representation of a list of strings to an actual list of strings
function stringToListWStrings(listAsString) {
  return listAsString
    .slice(1, -1)
    .split(', ')
    .map(d => d.slice(1, -1));
}
</script>

<!-- Cards with big numbers -->
<div class="grid grid-cols-4">
  <div class="card">
    <h2>y-axis <span class="muted">/ Criterium</span></h2>
    <span class="big">
      <select name="yaxis" id="criterium">
        <option value="scans">scans</option>
        <option value="PSMs">PSMs</option>
        <option value="peptides">peptides</option>
        <option value="peptidoforms">peptidoforms</option>        
      </select>
    </span>
  </div>
  <div class="card">
    <h2>FDR threshold</h2>
    <span class="big">      
      <select name="threshold" id="threshold">
        <option value="0.001">0.1 %</option>
        <option value="0.01" selected="selected">1 %</option>
        <option value="0.05">5 %</option>
        <option value="1">none</option>
      </select>
    </span>
  </div>
</div>

<div class="grid grid-cols-2">
  <div id='msamandaDiv' class="card"></div>
</div>

```js
const dataFiltered = FileAttachment(`data/compare_msamanda.tsv`).tsv({typed: true});
const dataUnfiltered = FileAttachment(`data/compare_msamanda_nofilter.tsv`).tsv({typed: true});
```

```js
function stringToNumericList(listAsString) {
  const listOfNumericValues = listAsString
    .substr(1, listAsString.length -2)
    .split(', ')
    .map((d) => Number(d));
  return listOfNumericValues
};

function stringToListWStrings(listAsString) {
  const listOfStrings = listAsString
    .substr(1, listAsString.length -2)
    .split(', ')
    .map((d) => d.substr(1, d.length-2));
  return listOfStrings
  };
async function fetchCSV(url) {
  const response = await fetch(url);
  const text = await response.text();
  return text;
}
```

```js
let countValue = document.getElementById('criterium').value;
let thresholdValue = document.getElementById('threshold').value;

async function handleInputChange(event) {
  const inputElement = event.target;
  const inputValue = inputElement.value;

  switch (inputElement.id) {
    case 'criterium':
      countValue = inputValue;
      break;
    case 'threshold':
      thresholdValue = inputValue;
      break;
  }

  console.log('criterium:', countValue);
  console.log('threshold:', thresholdValue);


  console.log(dataFiltered)
  console.log(thresholdValue != 1 ? dataUnfiltered : dataFiltered)


  const filteredData = (thresholdValue == 1 ? dataUnfiltered : dataFiltered)
    .filter(d => d.threshold == thresholdValue)
    .filter(d => d.criterium == countValue)
    .map(d => ({
      ...d,
      samples: stringToListWStrings(d.samples),
      counts: stringToNumericList(d.counts)
    }));

  console.log(filteredData)
  const colorScheme = ["#fc8d62","#ffd92f","#a6d854","#66c2a5","#8da0cb","#e78ac3"]
  const colorSchemeBorder = ["#e27043","#e0c23b","#97c64d","#5aab91","#798ab1","#d27cb1"]

  const methods = [
    { method: 'first', name: '0', color: colorScheme[0], lineColor: colorSchemeBorder[0], order: 1 },
    { method: 'MaxPrec1', name: '1', color: colorScheme[1], lineColor: colorSchemeBorder[1], order: 2 },
    { method: 'MaxPrec2', name: '2', color: colorScheme[2], lineColor: colorSchemeBorder[2], order: 3 },
    { method: 'MaxPrec3', name: '3', color: colorScheme[3], lineColor: colorSchemeBorder[3], order: 4 },
    { method: 'MaxPrec4', name: '4', color: colorScheme[4], lineColor: colorSchemeBorder[4], order: 5 },
    { method: 'MaxPrec5', name: '5', color: colorScheme[5], lineColor: colorSchemeBorder[5], order: 6 },
  ];

  var y1Data = {}, xData = {}, traces = [];

  methods.forEach(m => {
    const chartData = filteredData.find(d => d.method == m.method);
    y1Data[m.method] = chartData.counts;
    xData[m.method] = chartData.samples;

    // Create trace for the bar chart
    traces.push({
      x: xData[m.method],
      y: y1Data[m.method],
      name: m.name,
      type: 'bar',
      marker: {
        color: m.color,
        line: {
          color: m.lineColor,
          width: 1.5
        }
      }
    });
  });

  const layout = {
    barmode: 'group',
    bargap: 0.15,
    bargroupgap: 0.2,
    showlegend: true,
    // autosize: false,
    // width: 760,
    // height: 500,  
    legend: {
      title: {text: 'Max. number of allowed precursors'},
      y: -0.2,
      orientation: 'h',
    //   bordercolor: 'black',
    //   borderwidth: 1.5,
    },
    yaxis: {
      title: {
        text: `number of ${countValue}`,
        // text: `number of target PSMs`,
        standoff: 20,
        },                      
      // titlefont: {
      //     family: 'Arial, sans-serif',
      //     size: 18,
      //     color: 'grey'
      //     },
      // dtick: 25000,
      tick0: 0,
    },
    title: {
      text: `Number of identified ${countValue} (${thresholdValue*100}% FDR)`,
      // text: `number of target PSMs`,
      font: {
          family: 'Arial, sans-serif',
          size: 20,
          color: 'darkgrey',                
          },
      x: 0.5,
      y: 5,
      xref: 'paper',
      yref: 'paper',
      xanchor: 'center',
      yanchor: 'top',
    },                
    margin: {
      l: 100,
      r: 50,
      b: 100,
      t: 80,
      pad: 4
    },
  };
  msamandaDiv.fn = `compare_msamanda_${countValue}_${thresholdValue}`
  Plotly.newPlot('msamandaDiv', traces, layout);
}

// Add event listeners for input elements
document.getElementById('criterium').addEventListener('input', handleInputChange);
document.getElementById('threshold').addEventListener('input', handleInputChange);
```
