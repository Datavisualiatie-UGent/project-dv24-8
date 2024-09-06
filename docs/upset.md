---
theme: dashboard
title: upset plots
toc: false
---
<h1>
UpSet plots
</h1>

## visualizing intersections between pipelines


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


```js
const data = FileAttachment(`data/data_upset.csv`).csv({typed: true});
```

<div style="width: 80%;">
When comparing peptide identification across multiple pipelines, understanding which peptides are unique to a specific method and which are shared between pipelines is crucial for evaluating their performance. The UpSet plot for Sample S11 displays the intersection sizes between the four pipelines, highlighting how many peptides are shared or exclusive to each pipeline.
</div>
<div class="grid grid-cols-2">
  <div class="grid grid-cols-3">
    <div class="card">
      <h2>Sample</h2>
      <span class="big">S11</span>
    </div>
    <div class="card">
      <h2>FDR threshold</h2>
      <span class="big">1% </span>
    </div>
    <div class="card">
      <h2>Criterium</h2>
      <span class="big">peptides</span>
    </div>
  </div>
</div>


<div class="flexcontainer" style="width: 80%;">
  <div id='upsetChartExample' class="subchart" style="width: 70%;"></div>
  <div  id="horbarChartExample" class="subchart" style="width: 25%;"></div>
</div>



```js
const exampleData = data
    .filter(d => (d.sample=='S11')&&(d.threshold=='0.01'))
    .filter(d => (d.analysis=='default')&&(d.criterium=='peptides'))
    .filter(d => (d.value != 0))
    .sort((a,b) => Number(b.value) - Number(a.value))
    .sort((a,b) => b.variable.length - a.variable.length)

const xData = exampleData.map((d) => d.variable)
const yData = exampleData.map((d) => Number(d.value))

  let colors = [];

  xData.forEach((d) => {
    colors.push('rgba(62,62,62,1)')
    // if (d.includes('A') && !d.includes('B')) {
    //   if (d.includes('C') && !d.includes('D')) {
    //     colors.push('purple')
    //   } else {
    //   colors.push('red')
    //   }} else {
    //   if (d.includes('C') && !d.includes('D')) {
    //     colors.push('cornflowerblue')
    //   } else {
    //     colors.push('rgba(62,62,62,1)')
    //   }
    // }
  })
  console.log(colors)

  const barsData = [{
    x: xData,
    y: yData,
    type: 'bar',                  
    marker: {
      color: colors,
    }
  }];

  const heightSubplotCircles = 100, widthSubplotCircles = 660;
  const cy = -1*Math.max(...yData)/15;

  let circles = [], textAnnotations = [], percentages = [];

  circles.push({
    type: 'rect',
    xref: 'paper',
    yref: 'y',
    x0: -0.2,
    x1: 1,
    y0: cy,
    y1: 8*cy,
    fillcolor: 'rgba(255,255,255,1)',
    line: {width: 0},
  });
  
  for (let i = 0; i < exampleData.length; i++) {
    for (let j = 1; j < 5; j++) {
      var currentgroup = xData[i];
      circles.push({
        type: 'circle',
        xref: 'x',
        x0: i-0.3,
        x1: i+0.3,                
        yref: 'y',
        y0: (j-1)*cy + (j)*cy,
        y1: (j-1)*cy + (j+1)*cy,
        fillcolor:  currentgroup.includes('ABCD'[j-1]) ? colors[i] : 'lightgrey',
        line: {
          color:  currentgroup.includes('ABCD'[j-1]) ? colors[i] : 'lightgrey',
        }
      })
    };
    // add line through circles
    var yStart = 'ABCD'.indexOf(currentgroup[0]) + 1
    var yEnd = 'ABCD'.indexOf(currentgroup[currentgroup.length - 1]) + 1
    circles.push({
      type: 'line',
      xref: 'x',
      yref: 'y',
      x0: i,
      x1: i,
      y0: (yStart-1)*cy + (yStart)*cy,
      y1: (yEnd-1)*cy + (yEnd+1)*cy,
      line: { color: colors[i], width: 5},
    })
  }


  let sum = 0;

  for (let i = 0; i < yData.length; i++ ) {
    sum += yData[i];
  }

  let tmp = [];

  for (let i = 0; i < yData.length; i++ ) {
    
    textAnnotations.push({
      x: i,
      y: yData[i] - cy*1.3,
      xref: 'x',
      yref: 'y',
      text: `${yData[i]}`,
      showarrow: false,
      // font: {
      //     color: 'cornflowerblue'
      // }
    }),
    textAnnotations.push({
      x: i,
      y: yData[i] - cy*0.6,
      xref: 'x',
      yref: 'y',
      text:  `(${(yData[i]/sum*100) > 1 ? (yData[i]/sum*100).toFixed(1): '<1'}%)`,
      showarrow: false,
      // font: {
      //     color: 'cornflowerblue'
      // }
    });
    tmp.push(`${(yData[i]/sum*100) > 1 ? (yData[i]/sum*100).toFixed(1): '<1'}%`)
  }

  console.log(tmp)
  textAnnotations.push({
    x: -0.1,
    y: Math.max(...yData)/2,
    xref: 'paper',
    yref: 'y',            
    text: 'intersection size',
    showarrow: false,
    textangle: -90,
    font: {
      size: 16
    }
  })

  console.log(textAnnotations)

  const pipelines = [
      { abb: "D", full: "MS²Rescore w chimeric spectra"},
      { abb: "C", full: "MS²Rescore w/o chimeric spectra"},
      { abb: "B", full: "MSAmanda w chimeric spectra"},
      { abb: "A", full: "MSAmanda w/o chimeric spectra"},
  ];

  for (let j = 1; j < 5; j++) {
    textAnnotations.push({
      xref: 'x',
      x: -0.5,
      yref: 'y',
      y: (j-1)*cy + (j+0.5)*cy,
      text: `${pipelines[(4-j)].full}`,
      showarrow: false,
      xanchor: 'right',
    });
  };

  let countData = [], catData = [], textAnnotationsHorizontalBars = [];

  pipelines.forEach((d, j) => {
    let count=0;
    exampleData.forEach((groups) => {
      count += groups.variable.includes(d.abb) ? Number(groups.value) : 0
    })
    d.count = count;
    d.percentage = count/sum*100;
    countData.push(count)
    catData.push(d.abb)
  });

  const xAxisOffset = Math.max(...countData)*1.01

  pipelines.forEach((d, j) => {
    textAnnotationsHorizontalBars.push({
      xref: 'x',
      x: xAxisOffset,
      yref: 'y',
      y: j,
      text: `${d.count} (${(d.percentage).toFixed(1)}%)`,
      xanchor: 'left',
      showarrow: false,
    })
  })

  const horbarsData = [{
    x: countData,
    y: catData,
    type: 'bar',
    orientation: 'h',              
    marker: {
      color: 'rgba(62,62,62,1)',
      // line: {
      //   color: 'rgb(84, 130, 214)',
      //   width: 1.5
      // }
    }
  }];

  const layout = {
    xaxis: {
      showgrid: false,
      showticklabels: false,
      range: [-0.5, xData.length - 0.5],
    },
    yaxis: {
      showgrid: true,
      tickformatstops: {
        dtickrange: [0, null],
      },
      zeroline: false,
      gridcolor: 'lightgrey',
      nticks: 6,
    },
    height: 900,
    margin: {
      l: 220,
      r: 0,
      b: 200,
      t: 80,
      pad: 4
    },
    separators: ',.',
    shapes: circles,
    annotations: textAnnotations,
  };

  const layoutHorizontalBarChart = {
    xaxis: {
    //   showgrid: false,
    //   showticklabels: false,
      range: [0, xAxisOffset],
      nticks: 2,
      zeroline: {
        color: 'lightgrey'
      },
      gridcolor: 'lightgrey',
      tick0: 0,
    },
    yaxis: {
      // showgrid: false,
      showticklabels: false,
      zeroline: true,
    },
    margin: {
      l: 10,
      r: 150,
      b: 190,
      t: 510,
      pad: 4
    },
    annotations: textAnnotationsHorizontalBars,
    };
  // upsetChart.fn = `tda_hist_${sampleValue}_${thresholdValue}`;
  Plotly.newPlot('upsetChartExample', barsData, layout);
  Plotly.newPlot('horbarChartExample', horbarsData, layoutHorizontalBarChart);

  var UpSetPlot = document.getElementById('upsetChartExample')
  // Handle hover event
  UpSetPlot.on('plotly_hover', function(data){
  var pn = '',
      tn = '',
      colorsNew = [];

  // Update the bar color when hovered
  for (var i = 0; i < data.points.length; i++){
    pn = data.points[i].pointNumber;
    tn = data.points[i].curveNumber;
    colorsNew = data.points[i].data.marker.color;
  };
  colorsNew[pn] = '#C54C82';

  var update = {'marker': {color: colorsNew, size: 16}};
  Plotly.restyle('upsetChartExample', update);

  // Now handle the shape (circle or line) hover effect
  var newShapes = [];
  var layoutShapes = UpSetPlot.layout.shapes;

  // Loop through layout shapes to find the corresponding shapes for the hovered point
  for (var shapeIndex = 0; shapeIndex < layoutShapes.length; shapeIndex++){
    var shape = layoutShapes[shapeIndex];

    if ((shape.x0 === pn - 0.3 && shape.fillcolor === 'rgba(62,62,62,1)') || (shape.x0 === pn)) {
      // Update shape color on hover
      newShapes.push({
        ...shape,
        fillcolor: '#C54C82',  // New hover color for circles
        line: {
          color: '#C54C82', 
          width: 5      // New hover color for lines
        }
      });
    } else {
      newShapes.push(shape);  // Keep other shapes unchanged
    }
  };

  // Apply the updated shapes to the plot layout
  Plotly.relayout('upsetChartExample', { shapes: newShapes });
});

// Handle unhover event
UpSetPlot.on('plotly_unhover', function(data){
  var pn = '',
      tn = '',
      colorsBack = [];

  // // Revert the bar color when unhovered
  for (var i = 0; i < data.points.length; i++){
    pn = data.points[i].pointNumber;
    tn = data.points[i].curveNumber;
  //   colors = data.points[i].data.marker.color;
  };
  colors[pn] = 'rgba(62,62,62,1)';  // Revert to original color

  var update = {'marker': {color: colors}};
  Plotly.restyle('upsetChartExample', update, [tn]);
  Plotly.relayout('upsetChartExample', { shapes: circles });
});

```

# Interactive

<div class="grid grid-cols-4" style="width: 80%;">
  <div class="card">
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
    <h2>analysis</h2>
    <span class="big">
      <select name="analysis" id="analysis" style="width: 80px; " >
          <option value='default' selected="selected">default</option>
          <option value='final'>final</option>
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
    <h2>criterium</h2>
    <span class="big">
      <select id="criterium" style="width: 120px;">
          <option value='psms'>PSMs</option>
          <option value='peptides' selected="selected">peptides</option>
      </select>
    </span>
  </div>
</div>

<div class="flexcontainer" style="width: 80%;">
  <div id='upsetChart' class="subchart" style="width: 70%;"></div>
  <div  id="horbarChart" class="subchart" style="width: 25%;"></div>
</div>



```js
let sampleValue = document.getElementById('sample').value;
let criteriumValue = document.getElementById('criterium').value;
let thresholdValue = document.getElementById('threshold').value;
let analysisValue = document.getElementById('analysis').value;

async function handleInputChange(event) {
  const inputElement = event.target;
  const inputValue = inputElement.value;

  switch (inputElement.id) {
    case 'sample':
      sampleValue = inputValue;
      break;
    case 'criterium':
      criteriumValue = `${inputValue}`;
      break;
    case 'threshold':
      thresholdValue = inputValue;
      break;
    case 'analysis': 
      analysisValue = inputValue;
      break;
  }


  console.log('Sample:', sampleValue, 'Method:', analysisValue, 'Threshold', thresholdValue, 'selection: ', criteriumValue);

  const filteredData = data
    .filter(d => (d.sample==sampleValue)&&(d.threshold==thresholdValue))
    .filter(d => (d.analysis==analysisValue)&&(d.criterium==criteriumValue))
    .filter(d => (d.value != 0))
    .sort((a,b) => Number(b.value) - Number(a.value))
    .sort((a,b) => b.variable.length - a.variable.length)


  console.log('filteredData: ', filteredData);
  
  const xData = filteredData.map((d) => d.variable)
  const yData = filteredData.map((d) => Number(d.value))

  let colors = [];

  xData.forEach((d) => {
    colors.push('rgba(62,62,62,1)')
    // if (d.includes('A') && !d.includes('B')) {
    //   if (d.includes('C') && !d.includes('D')) {
    //     colors.push('purple')
    //   } else {
    //   colors.push('red')
    //   }} else {
    //   if (d.includes('C') && !d.includes('D')) {
    //     colors.push('cornflowerblue')
    //   } else {
    //     colors.push('rgba(62,62,62,1)')
    //   }
    // }
  })
  console.log(colors)

  const barsData = [{
    x: xData,
    y: yData,
    type: 'bar',                  
    marker: {
      color: colors,
    }
  }];

  const heightSubplotCircles = 100, widthSubplotCircles = 660;
  const cy = -1*Math.max(...yData)/15;

  let circles = [], textAnnotations = [], percentages = [];

  circles.push({
    type: 'rect',
    xref: 'paper',
    yref: 'y',
    x0: -0.2,
    x1: 1,
    y0: cy,
    y1: 8*cy,
    fillcolor: 'rgba(255,255,255,1)',
    line: {width: 0},
  });
  
  for (let i = 0; i < filteredData.length; i++) {
    for (let j = 1; j < 5; j++) {
      var currentgroup = xData[i];
      circles.push({
        type: 'circle',
        xref: 'x',
        x0: i-0.3,
        x1: i+0.3,                
        yref: 'y',
        y0: (j-1)*cy + (j)*cy,
        y1: (j-1)*cy + (j+1)*cy,
        fillcolor:  currentgroup.includes('ABCD'[j-1]) ? colors[i] : 'lightgrey',
        line: {
          color:  currentgroup.includes('ABCD'[j-1]) ? colors[i] : 'lightgrey',
        }
      })
    };
    // add line through circles
    var yStart = 'ABCD'.indexOf(currentgroup[0]) + 1
    var yEnd = 'ABCD'.indexOf(currentgroup[currentgroup.length - 1]) + 1
    circles.push({
      type: 'line',
      xref: 'x',
      yref: 'y',
      x0: i,
      x1: i,
      y0: (yStart-1)*cy + (yStart)*cy,
      y1: (yEnd-1)*cy + (yEnd+1)*cy,
      line: { color: colors[i], width: 5},
    })
  }


  let sum = 0;

  for (let i = 0; i < yData.length; i++ ) {
    sum += yData[i];
  }

  let tmp = [];

  for (let i = 0; i < yData.length; i++ ) {
    
    textAnnotations.push({
      x: i,
      y: yData[i] - cy*1.3,
      xref: 'x',
      yref: 'y',
      text: `${yData[i]}`,
      showarrow: false,
      // font: {
      //     color: 'cornflowerblue'
      // }
    }),
    textAnnotations.push({
      x: i,
      y: yData[i] - cy*0.6,
      xref: 'x',
      yref: 'y',
      text:  `(${(yData[i]/sum*100) > 1 ? (yData[i]/sum*100).toFixed(1): '<1'}%)`,
      showarrow: false,
      // font: {
      //     color: 'cornflowerblue'
      // }
    });
    tmp.push(`${(yData[i]/sum*100) > 1 ? (yData[i]/sum*100).toFixed(1): '<1'}%`)
  }

  console.log(tmp)
  textAnnotations.push({
    x: -0.1,
    y: Math.max(...yData)/2,
    xref: 'paper',
    yref: 'y',            
    text: 'intersection size',
    showarrow: false,
    textangle: -90,
    font: {
      size: 16
    }
  })

  console.log(textAnnotations)

  const pipelines = [
      { abb: "D", full: "MS²Rescore w chimeric spectra"},
      { abb: "C", full: "MS²Rescore w/o chimeric spectra"},
      { abb: "B", full: "MSAmanda w chimeric spectra"},
      { abb: "A", full: "MSAmanda w/o chimeric spectra"},
  ];

  for (let j = 1; j < 5; j++) {
    textAnnotations.push({
      xref: 'x',
      x: -0.5,
      yref: 'y',
      y: (j-1)*cy + (j+0.5)*cy,
      text: `${pipelines[(4-j)].full}`,
      showarrow: false,
      xanchor: 'right',
    });
  };

  let countData = [], catData = [], textAnnotationsHorizontalBars = [];

  pipelines.forEach((d, j) => {
    let count=0;
    filteredData.forEach((groups) => {
      count += groups.variable.includes(d.abb) ? Number(groups.value) : 0
    })
    d.count = count;
    d.percentage = count/sum*100;
    countData.push(count)
    catData.push(d.abb)
  });

  const xAxisOffset = Math.max(...countData)*1.01

  pipelines.forEach((d, j) => {
    textAnnotationsHorizontalBars.push({
      xref: 'x',
      x: xAxisOffset,
      yref: 'y',
      y: j,
      text: `${d.count} (${(d.percentage).toFixed(1)}%)`,
      xanchor: 'left',
      showarrow: false,
    })
  })

  const horbarsData = [{
    x: countData,
    y: catData,
    type: 'bar',
    orientation: 'h',              
    marker: {
      color: 'rgba(62,62,62,1)',
      // line: {
      //   color: 'rgb(84, 130, 214)',
      //   width: 1.5
      // }
    }
  }];

  const layout = {
    xaxis: {
      showgrid: false,
      showticklabels: false,
      range: [-0.5, xData.length - 0.5],
    },
    yaxis: {
      showgrid: true,
      tickformatstops: {
        dtickrange: [0, null],
      },
      zeroline: false,
      gridcolor: 'lightgrey',
      nticks: 6,
    },
    // autosize: false,
    // width: 930,
    height: 900,
    margin: {
      l: 220,
      r: 0,
      b: 200,
      t: 80,
      pad: 4
    },
    separators: ',.',
    shapes: circles,
    annotations: textAnnotations,
  };

  const layoutHorizontalBarChart = {
    xaxis: {
    //   showgrid: false,
    //   showticklabels: false,
      range: [0, xAxisOffset],
      nticks: 2,
      zeroline: {
        color: 'lightgrey'
      },
      gridcolor: 'lightgrey',
      tick0: 0,
    },
    yaxis: {
      // showgrid: false,
      showticklabels: false,
      zeroline: true,
    },
    margin: {
      l: 10,
      r: 150,
      b: 190,
      t: 510,
      pad: 4
    },
    annotations: textAnnotationsHorizontalBars,
    };
  // upsetChart.fn = `tda_hist_${sampleValue}_${thresholdValue}`;
  Plotly.newPlot('upsetChart', barsData, layout);
  Plotly.newPlot('horbarChart', horbarsData, layoutHorizontalBarChart);

  var UpSetPlot = document.getElementById('upsetChart')
// Handle hover event
UpSetPlot.on('plotly_hover', function(data){
  var pn = '',
      tn = '',
      colorsNew = [];

  // Update the bar color when hovered
  for (var i = 0; i < data.points.length; i++){
    pn = data.points[i].pointNumber;
    tn = data.points[i].curveNumber;
    colorsNew = data.points[i].data.marker.color;
  };
  colorsNew[pn] = '#C54C82';

  var update = {'marker': {color: colorsNew, size: 16}};
  Plotly.restyle('upsetChart', update);

  // Now handle the shape (circle or line) hover effect
  var newShapes = [];
  var layoutShapes = UpSetPlot.layout.shapes;

  // Loop through layout shapes to find the corresponding shapes for the hovered point
  for (var shapeIndex = 0; shapeIndex < layoutShapes.length; shapeIndex++){
    var shape = layoutShapes[shapeIndex];

    if ((shape.x0 === pn - 0.3 && shape.fillcolor === 'rgba(62,62,62,1)') || (shape.x0 === pn)) {
      // Update shape color on hover
      newShapes.push({
        ...shape,
        fillcolor: '#C54C82',  // New hover color for circles
        line: {
          color: '#C54C82', 
          width: 5      // New hover color for lines
        }
      });
    } else {
      newShapes.push(shape);  // Keep other shapes unchanged
    }
  };

  // Apply the updated shapes to the plot layout
  Plotly.relayout('upsetChart', { shapes: newShapes });
});

// Handle unhover event
UpSetPlot.on('plotly_unhover', function(data){
  var pn = '',
      tn = '',
      colorsBack = [];

  // // Revert the bar color when unhovered
  for (var i = 0; i < data.points.length; i++){
    pn = data.points[i].pointNumber;
    tn = data.points[i].curveNumber;
  //   colors = data.points[i].data.marker.color;
  };
  colors[pn] = 'rgba(62,62,62,1)';  // Revert to original color

  var update = {'marker': {color: colors}};
  Plotly.restyle('upsetChart', update, [tn]);
  Plotly.relayout('upsetChart', { shapes: circles });
});
  console.log('done');
}

// Add event listeners to each input box
document.getElementById('sample').addEventListener('input', handleInputChange);
document.getElementById('criterium').addEventListener('input', handleInputChange);
document.getElementById('threshold').addEventListener('input', handleInputChange);
document.getElementById('analysis').addEventListener('input', handleInputChange);
```

