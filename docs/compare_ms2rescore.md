---
theme: dashboard
title: comparison MS²Rescore results
toc: false
---

<script src="https://cdn.plot.ly/plotly-latest.min.js"></script>

# Comparison MS²Rescore pipelines 





<!-- Cards with big numbers -->
<div class="grid grid-cols-4">
  <div class="card">
    <h2>Search depth</h2>
    <span class="big">
      <select name="depth" id="depth">
        <option value='first'>first</option>
        <option value='second'>second</option>
        <option value='all'>both</option>
      </select>
    </span>
  </div>
  <div class="card">
    <h2>y-axis <span class="muted">/ Criterium</span></h2>
    <span class="big">
      <select name="yaxis" id="criterium">
        <option value="scans">scans</option>
        <option value="PSMs">PSMs</option>
        <option value="peptidoforms">peptidoforms</option>
        <option value="peptides">peptides</option>
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
      </select>
    </span>
  </div>
  <div class="card">
    <h2>Version</h2>
    <span class="big">
      <select name="method" id="method">
        <option value="old" selected="selected">old</option>
        <option value="new">new</option>
      </select>
    </span>
  </div>
</div>

<div class="grid grid-cols-1">
  <div id='myDiv' class="card"></div>
</div>

```js
const dataOld = FileAttachment(`data/grouped_results_ms2rescore_old.tsv`).tsv({typed: true});
const dataNew = FileAttachment(`data/grouped_results_ms2rescore_new.tsv`).tsv({typed: true});
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
```

```js
let depthValue = document.getElementById('depth').value;
let countValue = document.getElementById('criterium').value;
let thresholdValue = document.getElementById('threshold').value;
let methodValue = document.getElementById('method').value;

async function handleInputChange(event) {
  const inputElement = event.target;
  const inputValue = inputElement.value;

  switch (inputElement.id) {
    case 'depth':
      depthValue = inputValue;
      break;
    case 'criterium':
      countValue = `${inputValue}`;
      break;
    case 'threshold':
      thresholdValue = inputValue;
      break;
    case 'method': 
      methodValue = inputValue;
      break;
  }

  console.log('depth:', depthValue, 'criterium:', countValue, 'threshold:', thresholdValue, 'method: ', methodValue);

  const filteredData = (methodValue == 'old' ? dataOld : dataNew)
    .filter((d) => d.threshold == thresholdValue)
    .filter((d) => d.chart == depthValue)
    .filter((d) => d.criterium == countValue);

  const sepData = filteredData.filter((d) => d.training == 'separate')[0];
  const togetherData = filteredData.filter((d) => d.training == 'together')[0];

  var ySeparate = stringToNumericList(sepData.counts);
  var xSeparate = stringToListWStrings(sepData.samples);
  var yTogether = stringToNumericList(togetherData.counts);
  var xTogether = stringToListWStrings(togetherData.samples);            


  var trace1 = {
    x: xTogether,
    y: yTogether,
    name: 'together',
    legendgroup: 'leftLegend',
    type: 'bar',                  
    marker: {
      color: '#6996c6',
      line: {
        color: '#4e79a7',
        width: 1.5
      }
    }
  };

  var trace2 = {
    x: xSeparate,
    y: ySeparate,
    name: 'separate',
    legendgroup: 'rightLegend',
    type: 'bar',
    marker: {
      color: '#f9a34d',
      line: {
        color: '#f28e2c',
        width: 1.5
      }
    }
  };

  var offsetArrows = Math.max(
    stringToNumericList(sepData.counts).sort((a,b) => b-a)[0],
    stringToNumericList(togetherData.counts).sort((a,b) => b-a)[0]
  ) * 0.08

  var arrows = [];
  var percentagesDelta = [];
  var annotationsText = [];
  function getDeltaPc(xOld, xNew) {
      return ((xNew - xOld)/xOld)*100;
  }

  // Loop to create shapes and annotations
  for (let i = 0; i < xTogether.length; i++) {
      let sign = ySeparate[i] >= yTogether[i] ? '+' : '';
      annotationsText.push(`${sign}${(getDeltaPc(yTogether[i],ySeparate[i])).toFixed(2)}%`);
      
      let lineLayout = {
        color: ySeparate[i] >= yTogether[i] ? 'rgb(10,180,10)' : 'rgb(255, 0, 0)', // Green for positive change, red for negative
        width: 1,
        dash: 'solid'
      }

      let yMax = Math.max(yTogether[i],ySeparate[i]);

      arrows.push({
          type: 'line',
          x0: i -0.2,                                              
          x1: i + 0.2,    
          y0: yMax + offsetArrows,
          y1: yMax + offsetArrows,
          line: lineLayout
        },
        {
          type: 'line',
          x0: i -0.2,                                              
          x1: i - 0.2,    
          y0: yMax + (offsetArrows * 0.3),
          y1: yMax + offsetArrows,
          line: lineLayout
        },
        {
          type: 'line',
          x0: i + 0.2,                                              
          x1: i + 0.2,    
          y0: yMax + (offsetArrows * 0.3),
          y1: yMax + offsetArrows,
          line: lineLayout
        },
        {
          type: 'line',
          x0: i + 0.2,                                              
          x1: i + 0.25,    
          y0: yMax + (offsetArrows * 0.3),
          y1: yMax + (offsetArrows * 0.5),
          line: lineLayout
        },
        {
          type: 'line',
          x0: i + 0.2,                                              
          x1: i + 0.15,    
          y0: yMax + (offsetArrows * 0.3),
          y1: yMax +(offsetArrows * 0.5),
          line: lineLayout
        }
      );

      percentagesDelta.push({
          x: i,
          y: yMax + (offsetArrows *1.5),
          xref: 'x',
          yref: 'y',
          text: annotationsText[i],
          showarrow: false,
          font: {
              color: ySeparate[i] >= yTogether[i] ? 'rgb(10,180,10)' : 'rgb(255, 0, 0)' // Green for positive change, red for negative
          }
      });
  }

  let yAxisLabel = '';



  if (countValue !== 'PSMs') {
    yAxisLabel = `${countValue}`
  } else {if (depthValue == 'first') {
    yAxisLabel = `first search PSMs` 
  } else {
    if (depthValue =='second') {
      yAxisLabel = 'second search mPSMs'
    } else {
      yAxisLabel = 'all mPSMs'
    }
  }}

  var layout = {
    barmode: 'group',
    bargap: 0.15,
    bargroupgap: 0.1,
    showlegend: true,
    legend: {
      title: {
        text: 'Rescoring: '
      },
      // x: 0.01,
      // xanchor: 'left',
      // y: 0.98,
      y: 0.5,

      // bordercolor: 'black',
      // borderwidth: 1.5,
      // orientation: "h",
    },
    autosize: false,
    width: 760,
    height: 500,
    margin: {
      l: 80,
      r: 50,
      b: 100,
      t: 80,
      pad: 4
    },
    title: {
      text: `Number of identified ${countValue} (${depthValue}) at ${thresholdValue*100}% FDR`,
      font: {
        family: 'Arial, sans-serif',
        size: 24,
        color: 'darkgrey',                
    }},
    yaxis: {
      title: yAxisLabel,
      // title: '$\sqrt{(n_\text{c}(t|{T_\text{early}}))}$', 
      titlefont: {
        family: 'Arial, sans-serif',
        size: 16,
        color: 'black'
      },
      nticks: 6,
    },   
    shapes: arrows,
    annotations: percentagesDelta,
    };


  myDiv.fn = `ms2rescore_compare_${countValue} ${depthValue} ${thresholdValue}`;
  Plotly.newPlot('myDiv', [trace1, trace2], layout);
  }

// Add event listeners to each input box
document.getElementById('depth').addEventListener('input', handleInputChange);
document.getElementById('criterium').addEventListener('input', handleInputChange);
document.getElementById('threshold').addEventListener('input', handleInputChange);
document.getElementById('method').addEventListener('input', handleInputChange);
```
