<script>
    import { createEventDispatcher } from 'svelte';
    import ColorPicker from '../components/ColorPicker.svelte';
    import {
      Button,
      Content,
        Grid,
        Row,
        Column,
        TextArea,
        Slider,
    } from "carbon-components-svelte";
   
    export let pluginname;
    export let pluginemoji;
    export let name = "Dummy";
    export let description = "Dummy and dummy";
    export let color = "#DAAFE9";
    export let duration = 7;
    export let theme;
   
   const dispatch = createEventDispatcher();

   var disablebutton = true;

   $: if(!name || !description) {
    disablebutton = true;
   }
   else {disablebutton = false}
</script>

<Content>
    <Grid>
      <Row>
        <Column>
          <h1 style="text-align:center">{pluginemoji}</h1>
          <h2 style="text-align:center">{pluginname}</h2>
          <h5 style="text-align:center">Edit Fast</h5>
          <hr>
        </Column>
      </Row>
    </Grid>
    <TextArea rows={1} maxCount={20} labelText="Fast name" placeholder="Enter fast name..." bind:value={name}/>
        <TextArea rows={2} maxCount={200} labelText="Fast description" placeholder="Enter a description..." bind:value={description}/>
        <p>Color:</p>
        <ColorPicker bind:theme={theme} bind:value={color}></ColorPicker>
        <br>
        <Slider
            labelText="Hours"
            min={1}
            max={48}
            maxLabel="48"
            bind:value={duration}
        />
        <br>
        <Row>
          <Column>
              <br>
               <span><Button kind="secondary" on:click={()=>{dispatch("exitedit")}} style="float: left;">Exit</Button></span>
               <br>
          </Column>
        <Column>
            <br>
             <span><Button disabled={disablebutton} on:click={()=>{dispatch("savefast")}} style="float: right;">Save</Button></span>
             <br>
        </Column>
    </Row>    
  </Content>



  <style>
    h2 {
       margin: 0;
       padding: 0;
       font-size: 2.5em;
       font-weight: 400;
   }

 </style>

