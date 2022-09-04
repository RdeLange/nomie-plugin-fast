<script>
  import { onMount } from 'svelte';
  import "carbon-components-svelte/css/all.css";
  import nid from "./modules/nid/nid";
  import {
    Header,
    HeaderUtilities,
    HeaderGlobalAction,
    SkipToContent,
    Theme,
  } from "carbon-components-svelte";
  import Main from "./pages/main.svelte";
  import Info from "./pages/info.svelte";
  import Settings from "./pages/settings.svelte";
  import Edit from "./pages/main-edit.svelte";
  import MainExercise from "./pages/main-exercise.svelte";
  import SettingsAdjust from "carbon-icons-svelte/lib/SettingsAdjust.svelte";
  import Sun from "carbon-icons-svelte/lib/Sun.svelte";
  import Information from "carbon-icons-svelte/lib/Information.svelte";
  import { saveToLS, readFromLS } from './storage/storage';

  const pluginname = "Nomie Fast";
  const pluginemoji = "⏲";
  
  const plugin = new NomiePlugin({
        name: pluginname,
        emoji: pluginemoji,
        description: "Starter Template for Nomie6 Plugins",
        uses: ["createNote", "onLaunch", "getLocation","selectTrackables","getTrackable"],
        version: "0.5",
        addToCaptureMenu: true,
        addToMoreMenu: true,
        addToWidgets: true,
      }); 
  let inNomie = true;
  let isSideNavOpen = false;
  let theme = "g10";
  let mode = "hidden";
  let loading = true;
  let view = "main";
  let fasts;
  let config;
  let history;
  let amountofcards=0;
  let currentexercise = {};
      currentexercise.fasting = false;
  let currentexercisels = {};
      currentexercisels.fasting = false;    

  // Load init params
  function loadInitParams() {
    plugin.onUIOpened(async () => {
      mode = 'modal';
    });

    plugin.onWidget(() => {
      mode = "widget";
    });

    plugin.onRegistered(async () => {
      inNomie = true;
      loading = false;
      await plugin.storage.init()
      fasts = await plugin.storage.getItem('fasts') || [];
      history = await plugin.storage.getItem('history') || [];
      config = await plugin.storage.getItem('configuration') || {trackers:["none","none"],trackeroverrule:true,logentry:"Just finished <fast> exercise taking <duration> hours."};
      currentexercise = await plugin.storage.getItem('currentexercise') || {description: "",name:"",minutes: "", hours: "", day: "", duration: "", endtime: "", endday: "", fasting: false};
      saveToLS('currentfast', currentexercise);
      currentexercisels = JSON.parse(readFromLS('currentfast')) || {description: "",name:"",minutes: "", hours: "", day: "", duration: "", endtime: "", endday: "", fasting: false};
       
      if (plugin.prefs.theme == "light") {
        if(mode == "modal"){
          theme = "g10"}
        else {theme = "g10"}}
      else if (plugin.prefs.theme == "dark") {
        if(mode == "modal"){
          theme = "g90"}
        else {theme = "g100"}}  
      else {theme = "g10"} 
    })

    setTimeout(() => {
      if (loading) {
        inNomie = false;
      }
      else {
        amountofcards = fasts.length;
        if (currentexercise.fasting){
        view="mainexercise";
      }

      }
    }, 700);
  }

  // change theme
  function toggleTheme(){
    if (theme == "white"){
      theme = "g10"}
    else if (theme == "g10"){
      theme = "g80"}
    else if (theme == "g80"){
      theme = "g90"}
    else if (theme == "g90"){
      theme = "g100"}
    else {
      theme = "white"}
 }

 
//view main page
function showMain(){
  view = "main"
  window.scrollTo(0,0);
 }
 
 //view info page
 function showInformation(){
  view = "info"
  window.scrollTo(0,0);
 }

 //view settings page
 function showSettings(){
  view = "settings"
  window.scrollTo(0,0);
 }

//PLUGIN SPECIFIC CODE

let fastname = "";
let fastdescription = "Default Description"
let fastduration = 1;
let id;
let trackers = [];
let color = "";
let isEditMode = false;
let isAddMode = false;

 function showMainExercise(){
  fastname = "Default";
  fastdescription = "Default Description"
  fastduration = 1;
  view = "mainexercise"
  window.scrollTo(0,0);
 }

 function saveSettings(){
  showMain();
 }

 const storeCurrentFast = (event)=>{
  plugin.storage.setItem('currentexercise',event.detail);
  saveToLS('currentfast', event.detail);
  currentexercise = event.detail;
  currentexercisels = event.detail;
 }

 const storePrevious = (event)=>{
  var fastended = EventTarget.detail;
 if (history.length >= 7){
            history.unshift(parseInt(fastended));
            history.pop();
        } else {
            history.unshift(parseInt(fastended));
        }
  //plugin.storage.setItem('history',history);
 }

 // Set the edit mode
const editMode = (event) => {
  const fastId=event.detail;
		
		//find fast by id
		let fast = fasts.find(fast => fast.id === fastId);
		id = fast.id;
		fastname = fast.name;
		fastdescription = fast.description;
    fastduration= fast.duration;
    trackers = fast.trackers;
    color = fast.color;
		isEditMode = true;
    console.log('fast to edit', fastname);
    view="mainedit";
}

const startfast = event => {
  const fastId= event.detail;
		
		//find fast by id
		let fast = fasts.find(fast => fast.id === fastId);
		id = fast.id;
		fastname = fast.name;
		fastdescription = fast.description;
    fastduration= fast.duration;
    trackers = fast.trackers;
    color = fast.color;
    view="mainexercise";
    if (currentexercise.fasting){
      plugin.alert("Fasting in Progress", "There is currently a fasting in progress (see screen). Please end this fasting first before starting a new session");
    }
}

// Method to add a breathing    
const addfast = () => {
		
		if (isEditMode) {
			editfast(id, fastname, fastdescription, fastduration,trackers, color);
			
		} else {
			const fast = {
				id: nid(),
				name : fastname,
				description: fastdescription,
        duration: fastduration,
        trackers:trackers,
				color: color
			};
			fasts = fasts.concat(fast);
			resetAndSave(fasts);
		}
  view="main";  
}

// Method to delete a fast
const deletefast = event => {
    const id=event.detail;
		console.log('fast to delete', id);
		//find fast by id
		let index = fasts.findIndex(fast => fast.id === id);
		//remove fast
		fasts.splice(index, 1);
		fasts = [...fasts];
		console.log('fasts after delete', JSON.stringify(fasts));
		resetAndSave(fasts);
};

// Method to edit a fast
const editfast = (id, newName, newDescription, newDuration, newTrackers, newColor) => {
		console.log('fast to edit', fastname);
		//find fast by name
		let index = fasts.findIndex(fast => fast.id === id);
		//edit fast
		fasts[index].name = newName;
		fasts[index].description = newDescription;
    fasts[index].duration = newDuration;
    fasts[index].trackers = newTrackers;
    fasts[index].color = newColor;
		fasts = [...fasts];
		console.log('fasts after edit', fasts);
		resetAndSave(fasts);
};

// Method to reset the fast form
const reset = () => {
		id = '';
		fastname = '';
		color = '';
		fastdescription = '';
    fastduration = 10;
    trackers = ["None","None"];
		isEditMode = false;
}

// Method to reset and save
const resetAndSave = fasts => {
		reset();
		//saveToLS('breathings', breathings);
    plugin.storage.setItem('fasts', fasts);
    amountofcards = fasts.length;
}

// Method to save from Template
const saveTemplate = (event) => {
  const fast = {
				id: nid(),
				name : event.detail.detail.template.name,
				description: event.detail.detail.template.description,
        duration: event.detail.detail.template.duration,
        trackers:event.detail.detail.template.trackers,
				color: event.detail.detail.template.color
			};
			fasts = fasts.concat(fast);
			resetAndSave(fasts);

}

// Method to reset via settings modal
const resetAll = async (event) => {
  var res = {};
  res = await plugin.confirm('Resetting to default?', 'All your created exercises will be deleted');
    if (res.value) {
      fasts =[];
      resetAndSave(fasts);
      let configuration = {trackers:["none","none"],trackeroverrule:true,logentry:"Just finished <fast> exercise taking <duration> hours."};
      config = configuration;
      plugin.storage.setItem('configuration', config);  
    } 
}

const showfastinfo = event => {
  const fastId= event.detail;
		//find fast by id
		let fast = fasts.find(fast => fast.id === fastId);
  plugin.alert(fast.name, fast.description);
}

const newfast = () => {
    fastname = "";
		fastdescription= "";
    fastduration= 10;
    trackers=["None","None"];
		color= "#DAAFE9";
    isAddMode = true;
    view = "mainedit";
}

const logFast = (event) => {
  let note = event.detail;
  plugin.createNote(note);
}

 onMount(async () => {
  loadInitParams();
 })

</script>

{#if mode == "modal"  || mode =="widget"}
<Theme bind:theme />
{#if inNomie}
{#if mode == "modal"}
<Header company="Nomie6" platformName={pluginname} on:click={showMain}>
  <svelte:fragment slot="skip-to-content">
    <SkipToContent />
  </svelte:fragment>
  <HeaderUtilities>
    <HeaderGlobalAction aria-label="Settings" icon={SettingsAdjust} on:click={showSettings}/>
    <HeaderGlobalAction aria-label="Theme" icon={Sun} on:click={toggleTheme}/>
    <HeaderGlobalAction aria-label="Theme" icon={Information} on:click={showInformation}/>
  </HeaderUtilities>
</Header>


{#if view == "main"}
<Main pluginname={pluginname} pluginemoji={pluginemoji} bind:fastinginprogress={currentexercise.fasting} bind:amountofcards={amountofcards} bind:fasts={fasts} on:startfast={startfast} on:addnew={()=>{newfast()}} on:addbytemplate={saveTemplate} on:deletefast={deletefast} on:editfast={editMode} on:showfastinfo={showfastinfo} on:showexercise={showMainExercise}/>
{:else if view == "info"}
<Info pluginname={pluginname} pluginemoji={pluginemoji} on:exitinfo={showMain}/>
{:else if view == "settings"}
<Settings pluginname={pluginname} pluginemoji={pluginemoji} bind:theme={theme} on:resetall={resetAll} on:savetemplate={saveTemplate} bind:config plugin={plugin} on:savesettings={saveSettings} on:exitsettings={()=>{showMain()}}/>
{:else if view == "mainexercise"}
<MainExercise bind:mode={mode} pluginname={pluginname} pluginemoji={pluginemoji} plugin={plugin} bind:config={config} bind:current={currentexercise} bind:fastname={fastname} bind:fastduration={fastduration} bind:fastdescription={fastdescription} on:exitexercise={showMain} on:storecurrentfast={storeCurrentFast} on:storeprevious={storePrevious} on:logfast={logFast}/>
{:else if view == "mainedit"}
<Edit pluginname={pluginname} pluginemoji={pluginemoji} bind:theme={theme} on:addnew={addfast} on:savefast={addfast} on:exitedit={()=>{view="main"; isEditMode=false; isAddMode=false;}} bind:name={fastname} bind:description={fastdescription} bind:color={color} bind:duration={fastduration}></Edit>
{/if}
{:else if mode == "widget"}
<MainExercise bind:mode={mode} pluginname={pluginname} pluginemoji={pluginemoji} plugin={plugin} bind:config={config} bind:current={currentexercise} bind:fastname={fastname} bind:fastduration={fastduration} bind:fastdescription={fastdescription} on:exitexercise={showMain} on:storecurrentfast={storeCurrentFast} on:storeprevious={storePrevious} on:logfast={logFast}/>
{/if}
{/if}

{:else if !inNomie}
        <h1 style="text-align:center">{pluginemoji}</h1>
        <h2 style="text-align:center">{pluginname}</h2>
        <h5 style="text-align:center">This is a plugin for Nomie</h5>
        <hr>
{/if}
{#if loading}
<div class="startup">
<p>Loading....</p>
</div>
{/if}


<style>
  .startup {
    background-color: grey;
    height:100%;
    width:100%;
  }
</style>