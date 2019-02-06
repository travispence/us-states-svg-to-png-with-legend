<template>
  
  <div class="home">
      <div id="left" class="column">
        <div class="top-left">
            <span class="title">
                Legend Categories (Max 6)
            </span>
            <br>
            <br>
            <div class="flex-container wrap">
                <button class="btn btn-lg btn-vue-green" v-on:click="addCategory"> Add Category </button>
                <button class="btn btn-lg btn-vue-green" v-on:click="printSVG"> Download PNG </button>

            </div> 
        </div>
        <div class="bottom">
            <div style="width: 350px; margin: 1rem;" v-for="(category, idx) in categories" :key="idx">
                <div>
                    <label :for="idx"> Select Category</label>
                    <input type="radio" :id="idx" :value="idx + 1" v-model="selected">
                </div>
                <input style="width:100%;" type="text" :id="'label-' + idx" v-model="category.label">
                <br>
                <input type="text" :id="'color-' + idx" v-model="category.color">
                    
                <div class="md-chips flex-container wrap">
                    <div class="md-chip" v-for="(state, sidx) in category.states" :key="sidx">
                        <span>{{ state }}</span>
                        <button type="button" class="md-chip-remove" v-on:click="removeState(idx, sidx, state)">
                    </button>
                    </div>
                </div>
            </div>
        </div>
    </div>
    <div id="right" class="column">
        <div class="top-right"></div>
        <div class="bottom">
            <img class="svg" width="960" alt="US Map" src="../assets/us-states.svg" />
            <canvas id="canvas"></canvas>
        </div>
    </div>
  </div>
</template>

<script>
function resetStateFill(id) {
    document.querySelectorAll(`path#${id}`).forEach(function (path) {path.style.fill = '#d3d3d3'});
}

function fillColor(paths, hexColor) {
    document.querySelectorAll('path').forEach(function (path) {
        if (paths.indexOf(path.id) > -1) {
            path.style.fill = hexColor;
        }
    })
}

function createLabel(idx, color, text) {
    let colorBox = document.getElementById(`category_${idx}_color_box`);
    colorBox.style.strokeWidth = 1;
    colorBox.style.fill =  color;
    const labelText = document.getElementById(`category_${idx}_text_span`);
    labelText.textContent = text;
}


function drawInlineSVG(svgElement, ctx, callback) {
  var svgURL = new XMLSerializer().serializeToString(svgElement);
  var img  = new Image();
  img.onload = function(){
    ctx.drawImage(this, 0,0);
    callback();
    }
  img.src = 'data:image/svg+xml; charset=utf8, '+encodeURIComponent(svgURL);
}


export default {
  name: "home",
  data () {
    return {
    selected: 1,
    categories: [ {
      color: '#00695c',
      states: [],
      label: 'My First Category'
    }
    ]
  }
  },
  methods: {
      fillStates: function() {
        // resetStateFill();
        this.categories.forEach( (category, idx) => {
            fillColor(category.states, category.color)
            createLabel(idx + 1, category.color,  category.label );
        })
      },
      printSVG: function(){
          var myCanvas = document.getElementById("canvas");
          myCanvas.width = 1200;
          myCanvas.height= 600;
          var ctxt = myCanvas.getContext("2d");
          drawInlineSVG(document.querySelector('svg'), ctxt, function(canvas){
              const img = myCanvas.toDataURL();
            //   const newTab = window.open();
            var a = document.createElement('a');
            a.href = img;
            a.download = "output.png";
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            //   window.open(myCanvas.toDataURL());
            //   newTab.document.body.innerHTML = `<img src="${img}"></img>`
              })
      },
      addCategory: function addCategory() {
        this.categories.push({color: '#fafafa', states:[], label:'New Label' })
      },
      removeState: function(idx, sidx, id) {
        this.categories[idx].states.splice(sidx, 1);
        resetStateFill(id);
      }
      
  },
  watch : {
     categories: {
         handler() {
             this.fillStates();
         },
        deep: true
     }  
  },
  mounted () {
    const that = this;
    document.querySelectorAll('img.svg').forEach(function (img) {
        var imgID = img.id;
        var imgClass = img.className;
        var imgURL = img.src;

        fetch(imgURL).then(function (response) {
            return response.text();
        }).then(function (text) {

            var parser = new DOMParser();
            var xmlDoc = parser.parseFromString(text, "text/xml");

            // Get the SVG tag, ignore the rest
            var svg = xmlDoc.getElementsByTagName('svg')[0];

            // Add replaced image's ID to the new SVG
            if (typeof imgID !== 'undefined') {
                svg.setAttribute('id', imgID);
            }
            // Add replaced image's classes to the new SVG
            if (typeof imgClass !== 'undefined') {
                svg.setAttribute('class', imgClass + ' replaced-svg');
            }

            // Remove any invalid XML tags as per http://validator.w3.org
            svg.removeAttribute('xmlns:a');

            // Check if the viewport is set, if the viewport is not set the SVG wont't scale.
            if (!svg.getAttribute('viewBox') && svg.getAttribute('height') && svg.getAttribute('width')) {
                svg.setAttribute('viewBox', '0 0 ' + svg.getAttribute('height') + ' ' + svg.getAttribute('width'))
                svg.removeAttribute('width')
                svg.removeAttribute('height')
            }

            // Replace image with new SVG
            img.parentNode.replaceChild(svg, img);
            

            that.fillStates();

            document.querySelectorAll('svg').forEach(function (svg, idx) {
                svg.addEventListener('click', function (el) {
                    
                    const id = el.target.id;
                    const invalid = id.indexOf('path') > -1 || id.indexOf('category') > -1 || id == '';
                    if (!invalid){
                        const selected = that.selected - 1;
                        const index = that.$data.categories[selected].states.indexOf(id) ;
                        if(index > -1){
                            that.$data.categories[selected].states = that.$data.categories[selected].states.filter(state => {
                                return state !== id;
                            })
                            resetStateFill(id);
                        } else {
                            that.$data.categories.forEach(category => {
                                category.states = category.states.filter(state => {
                                    return state !== id;
                                })
                            });
                            that.$data.categories[selected].states.push(id);
                        }       
                    }
                });
            })
        });
    });
  }
};
</script>

<style scoped>
#canvas {
    visibility: hidden;
}
.row {
    display: flex;
    flex-wrap: wrap;
}

.row > .card {
    margin: 1rem;
}
.card {
  /* Add shadows to create the "card" effect */
  box-shadow: 0 4px 8px 0 rgba(0,0,0,0.2);
  transition: 0.3s;
}

/* On mouse-over, add a deeper shadow */
.card:hover {
  box-shadow: 0 8px 16px 0 rgba(0,0,0,0.2);
}

.title{
    font-size: 1.5rem;
}


.btn {
  background-color: transparent;
  outline: 0;
  -moz-transition: background-color 0.3s ease;
  -o-transition: background-color 0.3s ease;
  -webkit-transition: background-color 0.3s ease;
  transition: background-color 0.3s ease;
  font-family: 'Ubuntu', sans-serif;
}

.btn-lg {
  padding: 10px 37.5px 10px 37.5px;
  font-size: 1.2em;
}


.btn-vue-green {
  border: 2px solid #42b983;
}
.btn-vue-green:hover {
  background-color: #42b983;
}

/* Flex Container */
.flex-container {
  padding: 0;
  margin: 0;
  list-style: none;
  -ms-box-orient: horizontal;
  display: -webkit-box;
  display: -moz-box;
  display: -ms-flexbox;
  display: -moz-flex;
  display: -webkit-flex;
  display: flex;
}
.wrap    { 
  -webkit-flex-wrap: wrap;
  flex-wrap: wrap;
}  

/* Chips */
.md-chip {
  display: inline-block;
  background: #e0e0e0;
  padding: 0 12px;
  border-radius: 32px;
  font-size: 13px;
}
.md-chip.md-chip-hover:hover {
  background: #ccc;
}

.md-chip-clickable {
  cursor: pointer;
}

.md-chip, .md-chip-icon {
  height: 32px;
  line-height: 32px;
}

.md-chip-icon {
  display: block;
  float: left;
  background: #009587;
  width: 32px;
  border-radius: 50%;
  text-align: center;
  color: white;
  margin: 0 8px 0 -12px;
}

.md-chip-remove {
  display: inline-block;
  background: #aaa;
  border: 0;
  height: 20px;
  width: 20px;
  border-radius: 50%;
  padding: 0;
  margin: 0 -4px 0 4px;
  cursor: pointer;
  font: inherit;
  line-height: 20px;
}
.md-chip-remove:after {
  color: #e0e0e0;
  content: 'x';
}
.md-chip-remove:hover {
  background: #999;
}
.md-chip-remove:active {
  background: #777;
}

.md-chips {
  padding: 12px 0;
}
.md-chips .md-chip {
  margin: 0 5px 3px 0;
}

.md-chip-raised {
  box-shadow: 0 2px 2px 0 rgba(0, 0, 0, 0.14), 0 1px 5px 0 rgba(0, 0, 0, 0.12), 0 3px 1px -2px rgba(0, 0, 0, 0.2);
}
/* Material Radio */
@keyframes ripple {
  0% {
    box-shadow: 0px 0px 0px 1px rgba(0, 0, 0, 0);
  }
  50% {
    box-shadow: 0px 0px 0px 15px rgba(0, 0, 0, 0.1);
  }
  100% {
    box-shadow: 0px 0px 0px 15px rgba(0, 0, 0, 0);
  }
}
.md-radio {
  margin: 16px 0;
}
.md-radio.md-radio-inline {
  display: inline-block;
}
.md-radio input[type="radio"] {
  display: none;
}
.md-radio input[type="radio"]:checked + label:before {
  border-color: #337ab7;
  animation: ripple 0.2s linear forwards;
}
.md-radio input[type="radio"]:checked + label:after {
  transform: scale(1);
}
.md-radio label {
  display: inline-block;
  height: 20px;
  position: relative;
  padding: 0 30px;
  margin-bottom: 0;
  cursor: pointer;
  vertical-align: bottom;
}
.md-radio label:before, .md-radio label:after {
  position: absolute;
  content: '';
  border-radius: 50%;
  transition: all .3s ease;
  transition-property: transform, border-color;
}
.md-radio label:before {
  left: 0;
  top: 0;
  width: 20px;
  height: 20px;
  border: 2px solid rgba(0, 0, 0, 0.54);
}
.md-radio label:after {
  top: 5px;
  left: 5px;
  width: 10px;
  height: 10px;
  transform: scale(0);
  background: #337ab7;
}
/* flex layout */
html {
    height: 100%;
}
.home {
    display: flex;
}
body {
    height: 100%;   
    overflow: hidden;  /*makes the body non-scrollable (we will add scrolling to the sidebar and main content containers)*/
    margin: 0px;  /*removes default style*/
    /* display: flex;  enables flex content for its children */
    box-sizing: border-box;
}

.column {
    height: 100%;  /*allows both columns to span the full height of the browser window*/
    display: flex;
    flex-direction: column;  /*places the left and right headers above the bottom content*/
}

#left {
    flex-shrink: 0;  /*makes sure that content is not cut off in a smaller browser window*/
}
#right {
    flex-grow: 1;
}
.top-left {
    flex-shink: 0;
}

.top-right {
    flex-shrink: 0;
    display: inline-flex;
}

ul {
    display: inline-flex;
    list-style: none;
}

.bottom {
    flex-grow: 1;  /*ensures that the container will take up the full height of the parent container*/
    overflow-y: auto;  /*adds scroll to this container*/
}
</style>
