<template>
  <div class="home">
      <div style="display: flex; flex-direction: column; max-width: 960px; margin: 0 auto;">
          <div style="flex:2">
                <img class="svg" width="960" alt="US Map" src="../assets/us-states.svg" />
          </div>
            <div>
                <section>
                    <span class="title">
                        Legend Categories (Max 6)
                    </span>
                    <br>
                    <button class="btn btn-lg btn-ghost-blue" v-on:click="printSVG"> Download SVG </button>
                    <br>
                    <input v-model="selected" type="number" name="" min="1" :max="categories.length - 1" id="">
                </section>

                <div class="row">
                    <div class="card" v-for="(category, idx) in categories" :key="idx">
                        <div class="container">
                            <h3>
                                {{ category.label }} - <input type="radio"  :value="idx + 1" v-model="selected">
                            </h3>
                            <ul>
                                <li v-for="(state, sidx) in category.states" :key="sidx">
                                    {{ state }}
                                </li>
                            </ul>
                        </div>
                    </div>
                </div>
            </div>
      </div>
    <div style="width: 960px">
         <canvas id="canvas"></canvas>
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
    // console.log(`category_${idx}_color_box`)
    let colorBox = document.getElementById(`category_${idx}_color_box`);
    colorBox.style.strokeWidth = 1;
    colorBox.style.fill =  color;
    const labelText = document.getElementById(`category_${idx}_text_span`);
    labelText.textContent = text;
}

function add_category() {
    
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
      states: ['Colorado', 'Nevada'],
      label: 'States I have Visited'
    },
    {
      color: '#fdd835',
      states: ['NorthDakota'],
      label: 'States I have Traveled Through'
    },
    {
      color: '#ab47bc',
      states: ['Louisiana'],
      label: 'States I have Not Yet Visited'
    },
    ]
  }
  },
  methods: {
      fillStates: function() {
        resetStateFill();
        this.$data.categories.forEach( (category, idx) => {
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
            // console.log(svg);

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
            that.$data.categories[0].states.push('Alaska')
            that.$data.categories.push({color: '#fefefe', states:[], label: 'Shitty states'})
            document.querySelectorAll('svg').forEach(function (svg, idx) {
                svg.addEventListener('click', function (el) {
                    
                    const id = el.target.id;
                    console.log(id);
                    const invalid = id.indexOf('path') > -1 || id.indexOf('category') > -1;
                    if (!invalid){
                        const selected = that.selected - 1;
                        const index = that.$data.categories[selected].states.indexOf(id) ;
                        if(index > -1){
                            that.$data.categories[selected].states = that.$data.categories[selected].states.filter(state => {
                                console.log(state);
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

/* Add some padding inside the card container */
.container {
  padding: 2px 16px;
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


.btn-ghost-blue {
  border: 2px solid #3498db;
}
.btn-ghost-blue:hover {
  background-color: #3498db;
}
.btn-ghost-blue:active {
  background-color: #217dbb;
}


</style>
