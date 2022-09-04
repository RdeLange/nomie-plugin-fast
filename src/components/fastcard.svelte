<script>
    import { createEventDispatcher } from 'svelte'; 
    import TrashCan from "carbon-icons-svelte/lib/TrashCan.svelte";
    import Edit from "carbon-icons-svelte/lib/Edit.svelte";
    import Information from "carbon-icons-svelte/lib/Information.svelte";
export let fast = {
        id: '', 
        name: '', 
        description: '',
        color: '',
        duration: [],
        trackers: []
    }
    export let deleteFast;
    export let editFast;
    export let startFast;
    export let showFastInfo;

const dispatch = createEventDispatcher();
var color1 = fast.color;
var color2 = ColorLuminance(color1, 10);
var bg = "linear-gradient(135deg, "+color2+", "+color1+")";
var r = (Math.random()*2)+4;
var speed = r.toString()+"s";


$: if(fast){
    color1 = fast.color;
    color2 = ColorLuminance(color1, 10);
    bg = "linear-gradient(135deg, "+color2+", "+color1+")";
    r = (Math.random()*2)+4;
    speed = r.toString()+"s";
}

function ColorLuminance(hex, lum) {

// validate hex string
hex = String(hex).replace(/[^0-9a-f]/gi, '');
if (hex.length < 6) {
    hex = hex[0]+hex[0]+hex[1]+hex[1]+hex[2]+hex[2];
}
lum = lum || 0;

// convert to decimal and change luminosity
var rgb = "#", c, i;
for (i = 0; i < 3; i++) {
    c = parseInt(hex.substr(i*2,2), 16);
    c = Math.round(Math.min(Math.max(0, c + (c * lum)), 255)).toString(16);
    rgb += ("00"+c).substr(c.length);
}

return rgb;
}

</script>

<div class="organic" style="background:{bg}; animation-duration:{speed};" on:click={startFast}>
    <div class="row">
        <div class="column"><div class="actions">
            <span on:click|preventDefault|stopPropagation={showFastInfo}><Information size={20} /></span>
            <span on:click|preventDefault|stopPropagation={editFast}><Edit size={20} /></span>
            <span on:click|preventDefault|stopPropagation={deleteFast}><TrashCan size={20} /></span>
    </div></div>
      </div> 
    <p style="text-align:center;font-weight:500;width:80%;margin: 0 auto;">{fast.name}</p>
  </div>
  
    
  <style>
  .organic {
  display: flex;
  flex-direction: column;
  justify-content: center;
  width: 130px;
  height: 100px;
  background: linear-gradient(135deg, #FF512F, #DD2476);
  border-radius: 41% 59% 41% 59% / 43% 45% 55% 57%;
  animation-name: organic;
  animation-duration: 4s;
  animation-iteration-count: infinite;
  cursor: pointer;
}

.actions {
        justify-content:center;
        display: flex;
    }

    .actions span {
        padding: 0 0 0 .5rem;
        cursor: pointer;
    }


    .column {
  float:none;
  width: 100%;
}

/* Clear floats after the columns */
.row:after {
  content: "";
  display: table;
  clear: both;
}

@keyframes organic {
    0% {border-radius: 41% 59% 41% 59% / 43% 45% 55% 57%}
    33% {border-radius: 30% 70% 30% 70% / 32% 30% 70% 68%;}
    66% {border-radius: 70% 30% 70% 30% / 68% 70% 30% 32%;}
    100% {border-radius: 41% 59% 41% 59% / 43% 45% 55% 57%}
}
</style>