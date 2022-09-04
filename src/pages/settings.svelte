<script>
    import { createEventDispatcher, onMount } from 'svelte'; 
    import nid from "../modules/nid/nid";
    import {
      Button,
      Checkbox,
      TextInput,
      Tile,
      Grid,
      Row,
      Column,
      Content
    } from "carbon-components-svelte";
    import List from "carbon-icons-svelte/lib/TextAlignJustify.svelte";
    import CheckboxIndeterminate from "carbon-icons-svelte/lib/CheckboxIndeterminate.svelte";
    import {fastTemplates} from "../fastingTemplates";
    export let plugin;
    export let config;
    export let theme;
   
    export let pluginname;
    export let pluginemoji;

   let open=true;
   
   const dispatch = createEventDispatcher();
   
   

    let trackeroverrule = true;
    let trackers = ["none","none"];
    let logentry = "";
    let trackersDisplay =["∅ None","∅ None"];
    let selectedTemplates =[];
    let TemplateArray = [];
    let exercisetemplates = [];

  
	// Set background
	let themecolor = "#E9E9E9";
	let themefont = "#161616"
	if(theme=="g10"){
		themecolor = "grey";
		themefont = "#161616"
	}
	else {themecolor = "lightgrey";
	themefont = "#F4F4F4"}

    $: if(config){
      initConfig();
    }
   
    const addTracker = async (id) => {
      const justOneTrackable = await plugin.selectTrackables('tracker',false);
      if(justOneTrackable) {
        trackers[id]=justOneTrackable[0].tracker.tag;
        trackersDisplay[id]=justOneTrackable[0].tracker.emoji+" "+justOneTrackable[0].tracker.label;
      }
      
    }

    const removeTracker = async (id) => {
      trackers[id] = "none";
      trackersDisplay[id] = "∅ None";

      
    }

    const reset = () => {
      // reset hobboes
      dispatch("resetall");
      // reset config
      // save all
      // re-inityse data
    }

    
    function exitSettings(){
        dispatch("exitsettings");
    }

    function saveSettings(){
      // code to save settings
        config.trackers = trackers;
        config.trackeroverrule = trackeroverrule;
        config.logentry = logentry;
        dispatch("savesettings");
        open = false;
    }

    async function initConfig(){
      trackeroverrule = config.trackeroverrule;
      trackers = config.trackers;
      logentry = config.logentry;
      //trackers
      let trackable1;
      let trackable2;
      if (trackers[0] != "none") {
      trackable1 = await plugin.getTrackable('#'+trackers[0]);}
      else { trackable1 = {trackable:{tracker:{label:"None",emoji:"∅"}}}};
      if (trackers[1] != "none") {
      trackable2 = await plugin.getTrackable('#'+trackers[1]);}
      else { trackable2 = {trackable:{tracker:{label:"None",emoji:"∅"}}}};
      trackersDisplay =[trackable1.trackable.tracker.emoji+" "+trackable1.trackable.tracker.label,trackable2.trackable.tracker.emoji+" "+trackable2.trackable.tracker.label];
      }

    onMount (() =>{
      initConfig();
     // initTemplates();

    })
    
</script>

<Content>
    <Grid>
      <Row>
        <Column>
          <h1 style="text-align:center">{pluginemoji}</h1>
          <h2 style="text-align:center">{pluginname}</h2>
          <h5 style="text-align:center">Plugin Settings</h5>
          <hr>
        </Column>
      </Row>
    </Grid>
    <Tile>
    <h4>Nomie Log Settings:</h4>
      <Checkbox disabled labelText="Overrule individual Tracker Alignment" bind:trackeroverrule />
      <h3>Fast Mood:</h3>
      <tr>
        <td style="vertical-align:middle; text-align:center;width:20px"><span on:click={()=>{removeTracker(0)}}><CheckboxIndeterminate size={32} style="color:{themecolor};cursor: pointer;display:table-cell;vertical-align:middle;margin-top:0px"></CheckboxIndeterminate></span></td>
        <td style="vertical-align:middle; text-align:center;width:20px"><span on:click={()=>{addTracker(0)}}><List size={32} style="color:{themecolor};cursor: pointer;display:table-cell;vertical-align:middle;margin-top:0px"></List></span></td>
        <td style="vertical-align:middle ;text-align:left;width:250px"><p style="display:inline;font-weight:300;text-align:left;vertical-align:middle;">{trackersDisplay[0]}</p></td>
     </tr>
     <h3>Fast Duration:</h3>
      <tr>
        <td style="vertical-align:middle; text-align:center;width:20px"><span on:click={()=>{removeTracker(1)}}><CheckboxIndeterminate size={32} style="color:{themecolor};cursor: pointer;display:table-cell;vertical-align:middle;margin-top:0px"></CheckboxIndeterminate></span></td>
        <td style="vertical-align:middle; text-align:center;width:20px"><span on:click={()=>{addTracker(1)}}><List size={32} style="color:{themecolor};cursor: pointer;display:table-cell;vertical-align:middle;margin-top:0px;"></List></span></td>
        <td style="vertical-align:middle ;text-align:left;width:250px"><p style="display:inline;font-weight:300;text-align:left;vertical-align:middle;">{trackersDisplay[1]}</p></td>
     </tr>
     <h3>Additional Log Entry Input:</h3>
     <TextInput bind:value={logentry} placeholder="Enter additional Log input..."  helperText="You can include a reference to the related Fast Exercise. For instance: I just did <fast> for <duration> hours."/>
    </Tile>
      <br>
      <Tile>
      <h4>Reset to initial setup:</h4>
      <h6>This will remove all exercises and reset settings</h6>
      <br>
      <Button on:click={reset}>Reset</Button>
    </Tile>
    <br>
    <Row>
      <Column>
          <br>
           <span><Button kind="secondary" on:click={exitSettings} style="float: left;">Exit</Button></span>
           <br>
      </Column>
    <Column>
        <br>
         <span><Button on:click={saveSettings} style="float: right;">Save&Exit</Button></span>
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

 h6 {
       margin: 0;
       padding: 0;
   font-size: 0.8em;
   font-weight: 300;
 }

 p {
       height: 45px;
       line-height: 45px;
       text-align: center;
     }
 h3 {
   font-size:1.2em;
   font-weight: 300;
 }

 </style>

