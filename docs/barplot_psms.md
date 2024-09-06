---
theme: dashboard
title: comparison MS²Rescore results
toc: false
---

<script src="https://cdn.plot.ly/plotly-latest.min.js"></script>

# Comparison number of identified PSMs




<!-- Cards with big numbers -->
<div class="grid grid-cols-4" style="width: 80%">
  <div class="card">
    <h2>FDR threshold</h2>
    <span class="big">      
      <select name="threshold" id="threshold">
        <option value="0.001">0.1 %</option>
        <option value="0.01" selected="selected">1 %</option>
        <option value="0.05">5 %</option>
        <option value="1.0">None</option>
      </select>
    </span>
  </div>
  <div class="card">
    <h2>Version</h2>
    <span class="big">
      <select id="version">
        <option value='default' selected="selected">start</option> 
        <option value='final'>adapted</option>
      </select>
    </span>
  </div>
  <div class="card">
    <h2>scoring method</h2>
    <span class="big">
      <select name="scoring" id="scoringMethod">
        <option value='msamanda'>MSAmanda</option> 
        <option value='ms2rescore' selected="selected">MS²Rescore</option>
      </select>
    </span>
  </div>
  <div class="card">
    <h2>Method</h2>
    <span class="big">
      <select name="method" id="method">
        <option value='first'>first</option>
        <option value='MaxPrec1'>max 1 prec</option>              
        <option value='MaxPrec2'>max 2 prec</option>
        <option value='MaxPrec3'>max 3 prec</option>
        <option value='MaxPrec4'>max 4 prec</option>
        <option value='MaxPrec5'>max 5 prec</option>
        <option value='all' selected="selected">all</option>
        <option value='second'>second (ms2rescore)</option>
        <option value='merged'>merged (ms2rescore)</option>
      </select>
    </span>
  </div>
</div>

<div class="grid grid-cols-1" style="width: 80%">
  <div id='DivBarplot' class="card"></div>
</div>

```js
const data = FileAttachment(`data/barplot_nr_of_mpsms.tsv`).tsv({typed: true});
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
let thresholdValue = document.getElementById('threshold').value;
let methodValue = document.getElementById('method').value;
let versionValue = document.getElementById('version').value;
let scoringValue = document.getElementById('scoringMethod').value;


// Event listener for input change
async function handleInputChange(event) {
  const inputElement = event.target;
  const inputValue = inputElement.value;

  switch (inputElement.id) {
    case 'threshold':
      thresholdValue = inputValue;
      break;
    case 'method':
      methodValue = inputValue;
      break;
    case 'scoringMethod':
      scoringValue = inputValue;
      break;
    case 'version':
      versionValue = inputValue;
      break;
  }

  console.log('threshold:', thresholdValue);
  console.log('method: ', methodValue);
  console.log('version: ', versionValue);
  console.log('scoring: ', scoringValue);


  var filteredData = data
    // .filter(d => d.sample == sampleValue)
    .filter(d => d.threshold == thresholdValue)
    .filter(d => d.version == versionValue)
    .filter(d => d.scoring == scoringValue)
    .filter(d => d.method == methodValue)
    .sort((a, b) => b[2]-a[2]);

  console.log(filteredData)

  let samples = [];

  filteredData.forEach((d) => {
    samples.push(d.sample)
  })
  samples = ['S03','S05','S07','S08','S11','F01','F06','F07'];
                
  const colors = ['red','darkorange','gold','limegreen','aquamarine','cornflowerblue', 'mediumpurple', 'hotpink']
  const colorsBorders = ['crimson','chocolate','goldenrod','darkgreen','mediumspringgreen','blue', 'indigo', 'mediumvioletred']

  let sampleData = [], traces = [], labelsBars = []

  for (let i=0; i < samples.length; i++) {
    var sample = samples[i]
    sampleData = filteredData.filter(d => d.sample == sample)

    let xValues = [], yValues = []


    for (let j = 2; j < 7; j++) {
      xValues.push(`${j}`);
      yValues.push(Number(sampleData[0][j]))

      let pc = Number(sampleData[0][j])/sampleData[0]['total']*100;

      pc = (pc < 1 && pc != 0) ?  `< 1` : `${Math.round(pc*10)/10}`  

      labelsBars.push({
        x: j - 0.38 + i*0.106,
        y: Number(sampleData[0][j])+300,
        text: `${Number(sampleData[0][j])} (${pc}%)`,
        xanchor: 'center',
        yanchor: 'bottom',
        textangle: -90,
        showarrow: false,
        font: {
              color: colorsBorders[i]
          }
      })
    };
    traces.push({
      x: xValues,
      y: yValues,
      name: sample, 
      type: 'bar',
      marker: {
      color: colors[i],
      line: {
        color: colorsBorders[i],
        width: 1.5
      }
    }
    })
    console.log(sample, yValues)
  }

  const layout = {
    autosize: false,
    // width: 860,
    width: 1060,
    height: 500,
    margin: {
      l: 50,
      r: 50,
      b: 100,
      t: 150,
      pad: 4
    },

    barmode: 'group',
    bargap: 0.15,
    bargroupgap: 0.2,

    
    
    xaxis: {
      range: [1.5,5.5],
      title: `Number of identified peptides in one MS2 spectrum (${thresholdValue*100}% FDR)`,
      tickvalues: [2, 3, 4, 5],
      tickangle: -90,
                      
    },

    yaxis: {
      // showticklabels: false,
      title: '# MS2 spectra',
      tickangle: -90,
      side: 'right'
      
    },
    // showlegend: false,
    showlegend: true,

    legend: {
      x: 1,
      xanchor: 'right',
      y: 1.01,
      bordercolor: 'black',
      borderwidth: 1.5,
    },

    annotations: labelsBars,

    // title: {
    //   text: `Number of MS2 spectra with mPSMs`,
    //   font: {
    //       // family: 'Arial, sans-serif',
    //       size: 20,
    //       color: 'darkgrey',                
    //       },
    //   x: 0.5,
    //   y: 5,
    //   xref: 'paper',
    //   yref: 'paper',
    //   xanchor: 'center',
    //   yanchor: 'top',
    // },  
  }


  DivBarplot.fn = `mPSMs_per_scan_${scoringValue}_${methodValue}_${versionValue}`
  Plotly.newPlot('DivBarplot', traces, layout);
}

// Add event listeners for input elements
document.getElementById('threshold').addEventListener('input', handleInputChange);
document.getElementById('method').addEventListener('input', handleInputChange);
document.getElementById('version').addEventListener('input', handleInputChange);
document.getElementById('scoringMethod').addEventListener('input', handleInputChange);
```

