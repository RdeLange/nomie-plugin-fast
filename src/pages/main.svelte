<script>
    import { createEventDispatcher } from 'svelte';
    import {
      Content,
        Grid,
        Row,
        Column,
        Tile,
    } from "carbon-components-svelte";
    import FastCard from "../components/fastcard.svelte";
    import AddCard from "../components/addcard.svelte";
   
    export let pluginname;
    export let pluginemoji;
    export let fasts = [];
    export let amountofcards;
    export let fastinginprogress;
    

   let open=true;
   
   const dispatch = createEventDispatcher();

   
    function deletefast(id){
      dispatch("deletefast",id)
    }

    function editfast(id){
      dispatch("editfast",id)
    }

    function startfast(id){
      dispatch("startfast",id)
    }

    function showfastinfo(id){
      dispatch("showfastinfo",id)
    }

    
</script>

<Content>
    <Grid>
      <Row>
        <Column>
          <h1 style="text-align:center">{pluginemoji}</h1>
          <h2 style="text-align:center">{pluginname}</h2>
          <h5 style="text-align:center">Loaded Succesfully</h5>
          <hr>
          {#if fastinginprogress}
          <h5 on:click={()=>{dispatch("showexercise")}}  style="text-align:center;cursor:pointer;">⚠️ Fasting in progress ⚠️</h5>
          {/if}
        </Column>
      </Row>
      <Row>
        <Column>
    <div class="container">    
      <div>
        <div class="fasts-list">
          {#if fasts.length === 0}
          <Tile>
            <p class="no-fasts">
              No Fast Exercises? Oh dear, please add one to start practicing. 
            </p>
            
          </Tile>
          {:else}
            {#each fasts as fast}
              <FastCard
                fast={fast}
                deleteFast={() => deletefast(fast.id)}
                editFast={() => editfast(fast.id)}
                startFast={() => startfast(fast.id)}
                showFastInfo={() => showfastinfo(fast.id)}
                />
            {/each}
          {/if}
          <AddCard bind:amountofcards={amountofcards} on:addnew={()=>{dispatch("addnew")}} on:addbytemplate={(event)=>{dispatch("addbytemplate",event)}}/>
          
  
        </div>
      </div>
    </div>
      </Column>
      </Row>
    </Grid>
  </Content>



  <style>
    h2 {
       margin: 0;
       padding: 0;
       font-size: 2.5em;
       font-weight: 400;
   }

   .fasts-list {
  z-index: 0;
  width: 100%;
  display: flex;
  justify-content: space-around;
  flex-wrap: wrap;
}

.container {
		display: flex;
		justify-content: space-around;
		margin: 1rem auto auto auto;
	}
	@media screen and (max-width: 720px) {
		.container {
			flex-direction: column;
		}
	}

  .no-fasts {
		padding: 1em;
    	border: 1px solid;
    	border-radius: 4px;
	}


 </style>

