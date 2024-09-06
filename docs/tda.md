---
theme: dashboard
title: upset plots
toc: false
---

<script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
<style>
#criterium-container {
  display: none;
}
#selectlogbase-container {
  display: none;
}
#select-ms2rescore-method-container {
  display: none;
}
select {
  width: 60%;
  padding: 4px 4px;
  border-radius: 4px;
  background-color: #f1f1f1;
  text-align: center;
}
.flexcontainer {
  background-color: #ffffff;
  display: flex; 
  flex-wrap: wrap;
  border-radius: 10px; 
  padding: 10px;
  /* margin-left: 40px; */
  margin-bottom: 4px;
  /* margin-right: 40px; */
  margin-top: 4px;
  opacity: 0.8;
  border-color: #131d01;
  border-width: 0px;
  width: 100%;
  border-style: solid;
  align-content: center;
}
.flexcolumn {
  background-color: #c0c0c0; 
  display: flex; 
  flex-wrap: wrap;
  border-radius: 10px; 
  padding: 10px;
  padding-top: 10px ;
  margin-left: 4px;
  margin-bottom: 4px;
  margin-right: 4px;
  margin-top: 4px;
}     
.inputbox {
  background-color: #f0f0f0; 
  width: 100%;
  border-radius: 10px; 
  padding: 10px;
  margin-left:2px;
  margin-bottom: 2px;
  margin-right: 2px;
  margin-top: 2px;
}
.subchart{
  background-color: #f0f0f0; 
  width: 100%;
  padding: 0px;
  margin-left:20x;
  margin-bottom: 2px;
  margin-right: 0px;
  margin-top: 0px;        
}
</style>



# Target-decoy diagnostic plots
<div class="grid grid-cols-4">
  <div class="card">
    <h2>Sample</h2>
    <span class="big">F07</span>
  </div>
  <div class="card">
    <h2>Method</h2>
    <span class="big">MSAmanda</span>
  </div>
  <div class="card">
    <h2>Rank</h2>
    <span class="big">1</span>
  </div>
  <div class="card">
    <h2>Other</h2>
    <span class="big">Log2-transformation</span>
  </div>
</div>

<div class="grid grid-cols-2">
  <div id='histogramChartF07' class="card" ></div>
  <div  id="ppDivF07" class="card" ></div>
</div>


```js
const data_cutoff_scores = FileAttachment('data/cutoff_scores.csv').csv({typed: true});
const dataPPrank1 = FileAttachment('data/pp_info_rank1.csv').tsv({typed: true});
const dataPPallranks = FileAttachment('data/pp_info_allranks.csv').tsv({typed: true});
const data = FileAttachment("data/td_scores_rank1/S11_msamanda_first_scores.tsv").tsv({typed: true});
```

```js
const dataF07 = FileAttachment("data/td_scores_rank1/F07_msamanda_first_scores.tsv").tsv({typed: true});
const dataF07pp = dataPPrank1.filter(d => d.sample === "F07")
const data_cutoff_F07 = data_cutoff_scores.filter(d => d.sample === "F07")
```


```js
const filteredCutoffScores = data_cutoff_scores
    .filter((d) => (d.sample == "F07") && (d.method == "two_prec") && (d.scoring == "msamanda"))[0];

  var cutoff1, cutoff01, cutoff5, targetScores, decoyScores;

  var baseLogValue = 2
  var translationValue = getBaseLog(baseLogValue, filteredCutoffScores.pc1);
  
  cutoff01 = getBaseLog(baseLogValue, filteredCutoffScores.pc01) - translationValue;                                
  cutoff1 = getBaseLog(baseLogValue, filteredCutoffScores.pc1) - translationValue;
  cutoff5 = getBaseLog(baseLogValue, filteredCutoffScores.pc5) - translationValue;

  [targetScores, decoyScores] = [
    stringToNumericList(data.filter(d => d.type=="targets")[0].scores).map(d => getBaseLog(baseLogValue, d)-translationValue), 
    stringToNumericList(data.filter(d => d.type=="decoys")[0].scores).map(d => getBaseLog(baseLogValue, d)-translationValue)
    ];
  
  var customXAxisTitle = `log<sub>2</sub>(MSAmanda score)`


  var [xMinTargets, xMaxTargets] = [targetScores[0], targetScores[targetScores.length-2]];
  var [xMinDecoys, xMaxDecoys] = [decoyScores[0], decoyScores[decoyScores.length-2]];

  var zoomCheckbox = document.getElementById('zoom');
  var xMin = -5,
   xMax = 3;


  const xRange = [xMin, xMax];
  var binSize = 0.05
  
  console.log('xmin targets: ', xMinTargets, ' - ', 'xmax targets: ', xMaxTargets);
  console.log('xmin decoys: ', xMinDecoys, '-' , 'xmax decoys: ', xMaxDecoys);
  console.log('bin size:', binSize);     



  const traceDecoys = {
    x: decoyScores,
    name: 'decoys',
    autobinx: false,
    histnorm: "count",
    marker: {
      color: "rgba(255, 100, 102, 0.7)",
      line: {
        color: "rgba(255, 100, 102, 1)",
        width: 1
      }
    },
    opacity: 0.75,
    type: "histogram",
    xbins: {
      end: xMax,
      size: binSize,
      start: xMin
    }
  };
  const traceTargets = {
    x: targetScores,
    autobinx: false,
    histnorm: "count",
    marker: {
      color: "rgba(100, 200, 102, 0.7)",
      line: {
        color: "rgba(100, 200, 102, 1)",
        width: 1
      }
    },
    name: "targets",
    opacity: 0.5,
    type: "histogram",
    xbins: {
      end: xMax,
      size: binSize,
      start: xMin
    }
  };    
  const layout = {
    title: 'Score distributions',
    margin: {
      l: 50,
      r: 50,
      b: 70,
      t: 100,
      pad: 4
    },
    bargap: 0.05,
    bargroupgap: 0.2,
    barmode: "overlay",
    showlegend: true,
    legend: {
      x: 1,
      xanchor: 'right',
      y: 1.01,
      bordercolor: 'black',
      borderwidth: 0,
    },
    xaxis: { 
      title: customXAxisTitle,
      titlefont: {
          family: 'Arial, sans-serif',
          size: 18,
          color: 'grey'
          },
      range: [xMin, xMax], 
      // range: [-5,3],
      showgrid: false, 
      gridcolor: "#eee",
      griddash: "solid",
      gridwidth: "0.5"
    },
    yaxis: {
      title: {
        text: `Number of PSMs`, 
        standoff: 20,
        },                      
      titlefont: {
          family: 'Arial, sans-serif',
          size: 18,
          color: 'grey'
          },
      minexponent: 2,
      nticks: 4,
      ticklabelposition: 'inside top',
    },           
    shapes: [{
      type: 'line',
      x0: cutoff1,
      y0: 0,
      x1: cutoff1,
      y1: 1,
      yref: 'paper',
      line: {
        color: 'grey',
        width: 2,
        dash: 'dot'
      }},
    ],
    annotations: [{
      text: `1% FDR`,
      font: {color: '#6b0000',},
      xanchor: 'bottom',
      x: cutoff1,
      y: 1.07,
      yref: 'paper',
      xref: 'xaxis',
      ax: 40,
      ay: -20,
      showarrow: false,
    }]
  };
  


  const filteredDataPPF07 = dataPPrank1
    .filter((d) => (d.sample == "F07") && (d.method == "two_prec") && (d.scoring == "msamanda"))[0];

  console.log(filteredDataPPF07);

  var targetScores = filteredDataPPF07.targets;
  var cdf_targets = targetScores.substring(1, targetScores.length - 1).split(', ').map((d) => Number(d))

  var decoyScores = filteredDataPPF07.decoys;
  var cdf_decoys = decoyScores.substring(1, decoyScores.length - 1).split(', ').map((d) => Number(d))                 
  
  var trace1 = {
    x: cdf_decoys,
    y: cdf_targets,
    type: 'scatter',
    color: '#6996c6',
    zorder: 1
  };

  var trace2 = {
    x: [0, 1],
    y: [0, filteredDataPPF07.slope],
    name: 'pi0',
    type: 'scatter',
    mode: 'lines',   
    opacity: 0.75,
    line: {
      color: '#f28e2c',
      width: 5
    },
    zorder:0
  };

  var datab = [trace1, trace2];

  var layoutPP = {
    showlegend: false,
    margin: {
      l: 70,
      r: 50,
      b: 70,
      t: 50,
      pad: 4
    },
    title: `PP plot`, 
    xaxis: {
      title: "Fdp",
      titlefont: {
        family: 'Arial, sans-serif',
        size: 18,
        color: 'grey'
      },
      range: [0, 1.01]
    }, 
    yaxis: {
      title: "Ftp",
      titlefont: {
        family: 'Arial, sans-serif',
        size: 18,
        color: 'grey'
      },
      range: [0, 1.01]
    }, 
  };
  Plotly.newPlot('histogramChartF07', [traceDecoys, traceTargets], layout)
  Plotly.newPlot('ppDivF07', datab, layoutPP);
```



# Interactive
select sample to implement changes

<div class="grid grid-cols-4">
  <div class="card" >
    <h2>Sample</h2>
    <span class="big">
      <select id="sample" name="sample">
        <option value="F01">F01</option>
        <option value="F06">F06</option>
        <option value="F07" selected="selected">F07</option>
        <option value="S03">S03</option>
        <option value="S05">S05</option>
        <option value="S07">S07</option>
        <option value="S08">S08</option>
        <option value="S11">S11</option>
      </select>
    </span>
  </div>
  <div class="card">
    <span>
      <input type="checkbox" id="scoring" value="false" onclick="toggleSelectScoring()">
      <label for="scoring">MS²Rescore</label>
    </span>
  </div>
  <div class="card" id="select-msamanda-method-container">
    <h2>scoring</h2>  
    <span class="big">
      <select name="msamanda" id="msamandaMethod">
        <option value='first' selected="selected">first</option>
        <option value='second'>second</option>
        <option value='all'>all</option>
        <option value='one_prec'>max 1 prec</option>
        <option value='two_prec'>max 2 prec</option>
      </select>
    </span>
  </div>
  <div class="card" id="select-ms2rescore-method-container">
    <h2>scoring</h2>  
    <span class="big">
      <select  name="ms2rescore-method" id="ms2rescoreMethod">
            <option value='first' selected="selected">first</option>
            <option value='second'>second</option>
            <option value='all'>all</option>
            <option value='merged'>merged</option>
      </select>
    </span>
  </div>
  <div class="card">
    <h2>Rank</h2>
    <span class="big">
      <select id="selectrank" name="selectrank">
        <option value='rank1'>Rank 1 PSMs</option>
        <option value='allranks'>All PSMs</option>
      </select>
    </span>
  </div>
  <div class="card">
    <input type="checkbox" id="logtransform" value="true" onclick="toggleSelectBase()">
    <label for="logtransform">Log-transformation</label>
    <div id="selectlogbase-container">
      <label for="logbase">Base: &nbsp;</label>
      <select name="selectbase" id="logbase">
        <option value=2 selected="selected">2</option>
        <option value=10>10</option>
      </select>
    </div>
  </div>
  <div class="card">
    <span>
      <label for="nr_of_bins">Number of bins</label>
      <input type="number" id="nr_of_bins" name="quantity" min="10" max="500" value="300">
    </span>
  </div>
  <div class="card">

  </div>
  <div class="card">
    <span>
      <input type="checkbox" id="zoom" value=[-5,3]>
      <label for="zoom">Zoom &nbsp;</label>
    </span>
  </div>
</div>



<div class="grid grid-cols-2">
  <div id='histogramChart' class="card" ></div>
  <div  id="ppDiv" class="card" ></div>
</div>
<div class="grid grid-cols-2">
  <div  id="histogramChartWOdiff" class="card"></div>
  <div id='histogramChartSimple' class="card"></div>
</div>



<script>
  async function fetchCSV(url) {
    const response = await fetch(url);
    const text = await response.text();
    return text;
  }

  function stringToNumericList(listAsString) {
    const listOfNumericValues = listAsString
      .substr(2, listAsString.length -2)
      .split(', ')
      .map((d) => Number(d));
  return listOfNumericValues
  };

  function toggleSelectBase() {
    var logtransformCheckbox = document.getElementById('logtransform');
    var selectlogbaseContainer = document.getElementById('selectlogbase-container');

    if (logtransformCheckbox.checked) {
      selectlogbaseContainer.style.display = 'block';
    } else {
      selectlogbaseContainer.style.display = 'none';
    }
  };

  function toggleSelectScoring() {
    var ms2rescoreCheckbox = document.getElementById('scoring');

    if (ms2rescoreCheckbox.checked) {
        var selectMethodContainer =  document.getElementById("select-ms2rescore-method-container");
        var removeMethodContainer = document.getElementById("select-msamanda-method-container");
    } else {
      var selectMethodContainer =  document.getElementById("select-msamanda-method-container");
      var removeMethodContainer = document.getElementById("select-ms2rescore-method-container");
    };
    selectMethodContainer.style.display = 'block';
    removeMethodContainer.style.display = 'none';
  };

  function getBaseLog(base, value) {
    return Math.log(Number(value)) / Math.log(Number(base));
  }

  function processData(data, logTransform, translation, base) {
    var targetScores, decoyScores;

    if (logTransform) {
      decoyScores = data.filter(d => d.is_decoy === "True").map(d => getBaseLog(base, d.score)-translation);
      targetScores = data.filter(d => d.is_decoy === "False").map(d => getBaseLog(base, d.score)-translation);
    } else {
      decoyScores = data.filter(d => d.is_decoy === "True").map(d => Number(d.score));
      targetScores = data.filter(d => d.is_decoy === "False").map(d => Number(d.score));
    }
    return [targetScores, decoyScores];
  };

  function getMinMax(scores) {
    const sortedScores = scores.sort((a,b) => a-b);
    var scoreMin = sortedScores[0];
    var scoreMax = sortedScores[scores.length-1]; 
    return [scoreMin, scoreMax];
  }

  function SubtractHistogramCounts(data1, data2, min, max, numBins, binSize) {
    let subtractedCounts=[], medScores=[];

    for (let i=0; i < numBins; i++) {
      const binStart = min + i * binSize;
      const binEnd = binStart + binSize;
      subtractedCounts.push(data1.filter(d => d >= binStart && d < binEnd).length - data2.filter(d => d >= binStart && d < binEnd).length);
      medScores.push((binEnd+binStart)/2)
      }
    return [subtractedCounts, medScores];
  }
</script>

<!-- ```js 
const sampleInput = Inputs.select(data_cutoff_scores.map((d) => d.sample), {unique: true, sort: true, label: "sample:"});
const sampleValue = Generators.input(sampleInput);
```
<div class="card" style="display: flex; flex-direction: column; gap: 1rem;">${sampleInput}</div> -->

```js
// let binSize = document.getElementById('binsize').value;
const selectionRankValue = document.getElementById('selectrank').value;
const binsValue = document.getElementById('nr_of_bins').value;
let sampleValue = document.getElementById('sample').value;

async function handleInputChange_histogram(event) {
  const inputElement = event.target;
  const inputValue = inputElement.value;

  switch (inputElement.id) {
    case 'sample':
      sampleValue = inputValue;
      break;
  } 
  
  var scoringValue, methodValue, logbaseValue;

  if (document.getElementById('scoring').checked) {
    scoringValue = 'ms2rescore';
    methodValue = document.getElementById('ms2rescoreMethod').value;
    logbaseValue = false;
  } else {
    scoringValue = 'msamanda';
    methodValue = document.getElementById('msamandaMethod').value;
    logbaseValue = document.getElementById('logbase').value;
  }

  // console.log('Sample:', sampleValue, "&", 'Scoring:', scoringValue);
  // console.log('Method:', methodValue);
  // console.log('Number of bins:', binsValue);
  // console.log('Log base:', logbaseValue, "&", 'selection: ', selectionRankValue);

  
  // console.log(data_cutoff_scores)

  // parse CSV data w scores
  const filename = `./data/td_scores_${selectionRankValue}/${sampleValue}_${scoringValue}_${methodValue}_scores.tsv`;
  console.log(filename);  


  const csvData = await fetch(filename).then(response => response.text());
  const rows = csvData.trim().split('\n').map(row => row.split('\t'));
  const headers = rows[0];
  const data5 = rows.slice(1).map(row => {
    const obj = {};
    headers.forEach((header, i) => {
      obj[header] = row[i];
    });
    return obj;
  });

  console.log(data5)

  
  const filteredCutoffScores = data_cutoff_scores
    .filter((d) => (d.sample == sampleValue) && (d.method == methodValue) && (d.scoring == scoringValue))[0];

  const logtransformCheckbox = document.getElementById('logtransform');

  var cutoff1, cutoff01, cutoff5, targetScores, decoyScores;


  if ((logtransformCheckbox.checked) && (scoringValue == 'msamanda')) {
    const baseLogValue = document.getElementById('logbase').value;
    const translationValue = getBaseLog(baseLogValue, filteredCutoffScores.pc1);
    
    cutoff01 = getBaseLog(baseLogValue, filteredCutoffScores.pc01) - translationValue;                                
    cutoff1 = getBaseLog(baseLogValue, filteredCutoffScores.pc1) - translationValue;
    cutoff5 = getBaseLog(baseLogValue, filteredCutoffScores.pc5) - translationValue;

    // [targetScores, decoyScores] = processData(data, true, translationValue, baseLogValue);
    [targetScores, decoyScores] = [
      stringToNumericList(data.filter(d => d.type=="targets")[0].scores).map(d => getBaseLog(baseLogValue, d)-translationValue), 
      stringToNumericList(data.filter(d => d.type=="decoys")[0].scores).map(d => getBaseLog(baseLogValue, d)-translationValue)
      ];
    
    var customXAxisTitle = `log<sub>2</sub>(MSAmanda score)`
  } 
  else {
    cutoff01 = filteredCutoffScores.pc01;
    cutoff1 = filteredCutoffScores.pc1;
    cutoff5 = filteredCutoffScores.pc5;

    [targetScores, decoyScores] = [stringToNumericList(data.filter(d => d.type=="targets")[0].scores), stringToNumericList(data.filter(d => d.type=="decoys")[0].scores)];

    if (scoringValue == 'msamanda') {
      var customXAxisTitle = 'MSAmanda score'
    } else {
      var customXAxisTitle = 'MS<sup>2</sup>Rescore score'
    }
  }

  // console.log('cut-off scores: ', cutoff01, cutoff1, cutoff5);

  var [xMinTargets, xMaxTargets] = [targetScores[0], targetScores[targetScores.length-2]];
  var [xMinDecoys, xMaxDecoys] = [decoyScores[0], decoyScores[decoyScores.length-2]];

  var zoomCheckbox = document.getElementById('zoom');
  var xMin, xMax;

  
  if ((scoringValue == 'ms2rescore') || (document.getElementById('logtransform').checked)) {
    if (zoomCheckbox.checked) {
      xMin = -4;
      xMax = 4;
    } else {
      xMin = Math.floor(Math.min(xMinDecoys, xMinTargets));
      xMax = Math.ceil(Math.max(xMaxDecoys, xMaxTargets));
    }
  }
  else {
    if (zoomCheckbox.checked) {
      xMin = -20;
      xMax = xMaxDecoys;
    } else {
      xMin = -20;
      xMax = xMaxTargets;
    }
  }

  const xRange = [xMin, xMax];
  var binSize = (xMax-xMin)/binsValue
  
  console.log('xmin targets: ', xMinTargets, ' - ', 'xmax targets: ', xMaxTargets);
  console.log('xmin decoys: ', xMinDecoys, '-' , 'xmax decoys: ', xMaxDecoys);
  console.log('bin size:', binSize);     


  
  // Extract the bin counts from the trace objects
  const [yDataLineChart, xDataLineChart] = SubtractHistogramCounts(targetScores, decoyScores, xMin, xMax, binsValue ,binSize);

  const traceLineChart = {
      x: xDataLineChart,
      y: yDataLineChart,
      name: 'targets-decoys',
      type: 'scatter',
      line: {
        color: 'red',
        width: 2,
        shape: 'spline',
        smoothing: 2.5,
    }};

  const traceDecoys = {
    x: decoyScores,
    name: 'decoys',
    autobinx: false,
    histnorm: "count",
    marker: {
      color: "rgba(255, 100, 102, 0.7)",
      line: {
        color: "rgba(255, 100, 102, 1)",
        width: 1
      }
    },
    opacity: 0.75,
    type: "histogram",
    xbins: {
      end: xMax,
      size: binSize,
      start: xMin
    }
  };
  const traceTargets = {
    x: targetScores,
    autobinx: false,
    histnorm: "count",
    marker: {
      color: "rgba(100, 200, 102, 0.7)",
      line: {
        color: "rgba(100, 200, 102, 1)",
        width: 1
      }
    },
    name: "targets",
    opacity: 0.5,
    type: "histogram",
    xbins: {
      end: xMax,
      size: binSize,
      start: xMin
    }
  };    
  const layout = {
    margin: {
      l: 50,
      r: 50,
      b: 70,
      t: 150,
      pad: 4
    },
    bargap: 0.05,
    bargroupgap: 0.2,
    barmode: "overlay",
    showlegend: false,
    legend: {
      x: 1,
      xanchor: 'right',
      y: 1.01,
      bordercolor: 'black',
      borderwidth: 0,
    },
    xaxis: { 
      title: customXAxisTitle,
      titlefont: {
          family: 'Arial, sans-serif',
          size: 18,
          color: 'grey'
          },
      range: [xMin, xMax], 
      // range: [-5,3],
      showgrid: false, 
      gridcolor: "#eee",
      griddash: "solid",
      gridwidth: "0.5"
    },
    yaxis: {
      title: {
        text: `Number of PSMs`, 
        standoff: 20,
        },                      
      titlefont: {
          family: 'Arial, sans-serif',
          size: 18,
          color: 'grey'
          },
      minexponent: 2,
      nticks: 4,
      ticklabelposition: 'inside top',
    },
    title: {
      text: `${sampleValue} ${scoringValue} ${methodValue} - ${selectionRankValue}`,
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
    shapes: [{
      type: 'line',
      x0: cutoff1,
      y0: 0,
      x1: cutoff1,
      y1: 1,
      yref: 'paper',
      line: {
        color: 'grey',
        width: 2,
        dash: 'dot'
      }},
    ],
    annotations: [{
      text: `1% FDR`,
      font: {color: '#6b0000',},
      xanchor: 'bottom',
      x: cutoff1,
      y: 1.07,
      yref: 'paper',
      xref: 'xaxis',
      ax: 40,
      ay: -20,
      showarrow: false,
    }]
  };
  
  const layoutWOshapes = {
    margin: {
      l: 50,
      r: 50,
      b: 70,
      t: 150,
      pad: 4
    },
    bargap: 0.05,
    bargroupgap: 0.2,
    barmode: "overlay",
    showlegend: false,
    xaxis: { 
      title: customXAxisTitle,
      titlefont: {
          family: 'Arial, sans-serif',
          size: 18,
          color: 'grey'
          },
      range: [xMin, xMax], 
      // range: [-5,3],
      showgrid: true, 
      gridcolor: "#eee",
      griddash: "solid",
      gridwidth: "0.5",
    },
    yaxis: {
      title: {
        text: `Number of PSMs`, 
        standoff: 20,
        },                      
      titlefont: {
          family: 'Arial, sans-serif',
          size: 18,
          color: 'grey'
          },
      minexponent: 2,
      nticks: 4,
      ticklabelposition: 'inside top',
      showgrid: true,
    },
    title: {
      // text: `${sampleValue} ${scoringValue} ${methodValue} - ${selectionRankValue}`,
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
    }}
  

    const layoutAllCutoffs = {
      margin: {
      l: 50,
      r: 50,
      b: 70,
      t: 150,
      pad: 4
    },
    bargap: 0.05,
    bargroupgap: 0.2,
    barmode: "overlay",
    showlegend: false,
    xaxis: { 
      title: customXAxisTitle,
      titlefont: {
          family: 'Arial, sans-serif',
          size: 18,
          color: 'grey'
          },
      range: [xMin, xMax], 
      showgrid: true, 
      gridcolor: "#eee",
      griddash: "solid",
      gridwidth: "0.5"
    },
    yaxis: {
      title: {
        text: `Number of PSMs`, 
        standoff: 20,
        },                      
      titlefont: {
          family: 'Arial, sans-serif',
          size: 18,
          color: 'grey'
          },
      minexponent: 2,
      nticks: 4,
      ticklabelposition: 'inside top',
    },
    title: {
      // text: `${sampleValue} ${scoringValue} ${methodValue} - ${selectionRankValue}`,
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
      yanchor: 'bottom',

    },
    shapes: [{
      type: 'line',
      x0: cutoff01,
      y0: 0,
      x1: cutoff01,
      y1: 1,
      yref: 'paper',
      line: {
        color: 'grey',
        width: 1,
        dash: 'dot'
      }},
      {
      type: 'line',
      x0: cutoff1,
      y0: 0,
      x1: cutoff1,
      y1: 1,
      yref: 'paper',
      line: {
        color: 'grey',
        width: 2,
        dash: 'dot'
      }},
      {
      type: 'line',
      x0: cutoff5,
      y0: 0,
      x1: cutoff5,
      y1: 1,
      yref: 'paper',
      line: {
        color: 'grey',
        width: 1,
        dash: 'dot'
      }
    }
    ],
    annotations: [{
      text: `0.1% FDR<br>${Number(cutoff01).toFixed(1)}`,
      x: Number(cutoff01) + 0.2,
      y: 1.01,
      yref: 'paper',
      ax: 40,
      ay: -40,
    },
    {
      text: `1% FDR<br>${Number(cutoff1).toFixed(1)}`,
      font: {color: '#6b0000',},
      x: cutoff1,
      y: 1.01,
      yref: 'paper',
      xref: 'xaxis',
      ax: 0,
      ay: -40,
    },
    {
      text: `5% FDR<br>${Number(cutoff5).toFixed(1)}`,
      x: cutoff5 - 0.2,
      y: 1.01,
      yref: 'paper',
      ax: -40,
      ay: -40
    }]
  };

  console.log(dataPPrank1);
  console.log(dataPPallranks)


  const filteredDataPP = (selectionRankValue == "rank1" ? dataPPrank1 : dataPPallranks)
    .filter((d) => d.sample == sampleValue)
    .filter((d) => d.scoring == scoringValue)
    .filter((d) => d.method == methodValue);

  console.log(filteredDataPP);

  var targetScores = filteredDataPP[0].targets;
  var cdf_targets = targetScores.substring(1, targetScores.length - 1).split(', ').map((d) => Number(d))

  var decoyScores = filteredDataPP[0].decoys;
  var cdf_decoys = decoyScores.substring(1, decoyScores.length - 1).split(', ').map((d) => Number(d))                 
  
  var trace1 = {
    x: cdf_decoys,
    y: cdf_targets,
    type: 'scatter',
    color: '#6996c6',
    zorder: 1
  };

  var trace2 = {
    x: [0, 1],
    y: [0, filteredDataPP[0].slope],
    name: 'pi0',
    type: 'scatter',
    mode: 'lines',   
    opacity: 0.75,
    line: {
      color: '#f28e2c',
      width: 5
    },
    zorder:0
  };

  var datab = [trace1, trace2];

  var layoutPP = {
    showlegend: false,
    margin: {
      l: 70,
      r: 50,
      b: 70,
      t: 100,
      pad: 4
    },
    title: `PP plot<br> ${sampleValue} ${scoringValue} ${methodValue} ${selectionRankValue}`, 
    xaxis: {
      title: "Fdp",
      titlefont: {
        family: 'Arial, sans-serif',
        size: 18,
        color: 'grey'
      },
      range: [0, 1.01]
    }, 
    yaxis: {
      title: "Ftp",
      titlefont: {
        family: 'Arial, sans-serif',
        size: 18,
        color: 'grey'
      },
      range: [0, 1.01]
    }, 
  };
  ppDiv.fn = `pp_plot_${sampleValue}_${scoringValue}_${methodValue}_${selectionRankValue}`;
  histogramChart.fn = `tda_hist_${sampleValue}_${scoringValue}_log${logbaseValue}_${methodValue}_${selectionRankValue}`;
  Plotly.newPlot('histogramChart', [traceDecoys, traceTargets], layout)
  Plotly.newPlot('ppDiv', datab, layoutPP);
  Plotly.newPlot('histogramChartSimple', [traceDecoys, traceTargets, traceLineChart], layout);
  Plotly.newPlot('histogramChartWOdiff', [traceDecoys, traceTargets], layoutAllCutoffs);
  console.log('done');
}
document.getElementById('sample').addEventListener('input', handleInputChange_histogram);


```
