<script>
    import { createEventDispatcher } from 'svelte'; 
import { OverflowMenu, OverflowMenuItem } from "carbon-components-svelte";
import Add from "carbon-icons-svelte/lib/Add.svelte";
import {fastTemplates} from "../fastingTemplates";

export let amountofcards = 3;
let direction = "bottom";

$: if(amountofcards > 2) {
    direction = "top"
}
else {direction = "bottom"}


const dispatch = createEventDispatcher();

</script>

<div class="organic">
    <div class="actions">
    
    <h3>
    <OverflowMenu icon={Add} direction={direction}>

        <OverflowMenuItem text="Create Custom" on:click={()=>{dispatch("addnew")}}/>
        <hr>
        {#each fastTemplates as template }
           <OverflowMenuItem text={template.name} on:click={()=>{dispatch("addbytemplate",{template})}}/>
        {/each}
        
      </OverflowMenu>
    </h3>  
</div>
</div>
  
    
  <style>
  .organic {
  display: flex;
  flex-direction: column;
  justify-content: center;
  width: 130px;
  height: 100px;
  background: linear-gradient(135deg, #6f6f6f, #4b4b4b);
  border-radius: 41% 59% 41% 59% / 43% 45% 55% 57%;
  animation-name: organic;
  animation-duration: 4s;
  animation-iteration-count: infinite;
}

.actions {
        justify-content:center;
        display: flex;
    }



@keyframes organic {
    0% {border-radius: 41% 59% 41% 59% / 43% 45% 55% 57%}
    33% {border-radius: 30% 70% 30% 70% / 32% 30% 70% 68%;}
    66% {border-radius: 70% 30% 70% 30% / 68% 70% 30% 32%;}
    100% {border-radius: 41% 59% 41% 59% / 43% 45% 55% 57%}
}
</style>