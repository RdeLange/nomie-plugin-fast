<script>
  import { onMount } from 'svelte';
  import LibLoader from './components/LibLoadder.svelte';
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

  const pluginname = "Fasting Plugin";
  const pluginemoji = "⏲";
  var parent = "";
  
  const plugin = new NomiePlugin({
        name: pluginname,
        emoji: pluginemoji,
        avatar: "data:image/jpeg;base64,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",
        description: "Intermitted Fasting Timer",
        uses: ["createNote", "getLocation","selectTrackables","getTrackable"],
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
  let PlugiAapiUrl = "https://plugins.nomie.app/v1/nomie-plugin.js"; 

  // Load init params
  function loadInitParams() {
    parent = getParentUrl();
    plugin.onUIOpened(async () => {
      mode = 'modal';
    });

    plugin.onWidget(() => {
      if (plugin.prefs.theme == "light") {
        theme = "white"}
      else if (plugin.prefs.theme == "dark") {
        theme = "g100"}  
      else {theme = "g10"} 
      mode = "widget";
    });

    plugin.onRegistered(async () => {
      await plugin.storage.init()
      fasts = await plugin.storage.getItem('fasts') || [];
      history = await plugin.storage.getItem('history') || [];
      config = await plugin.storage.getItem('configuration') || {trackers:["none","none"],trackeroverrule:true,logentry:"Just finished <fast> exercise taking <duration> hours."};
      currentexercise = await plugin.storage.getItem('currentexercise') || {description: "",name:"",minutes: "", hours: "", day: "", duration: "", endtime: "", endday: "", fasting: false};
      saveToLS('currentfast', currentexercise);
      currentexercisels = JSON.parse(readFromLS('currentfast')) || {description: "",name:"",minutes: "", hours: "", day: "", duration: "", endtime: "", endday: "", fasting: false};
       
     // if (plugin.prefs.theme == "light") {
     //   theme = "g10"}
     // else if (plugin.prefs.theme == "dark") {
     //   theme = "g90"}  
     // else {theme = "g10"} 
    })
    amountofcards = fasts.length;
        if (currentexercise.fasting){
        view="mainexercise";}
   setTimeout(() => {
      inNomie = true;
      loading = false;
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

 // Get parent
 function getParentUrl() {
    var isInIframe = (parent !== window),
        parentUrl = null;

    var parentfound = null;
    
    if (isInIframe) {
        parentUrl = document.referrer;
    }

    if (parentUrl.includes("nomie") ) {
      parentfound = "Nomie"
    }
    else {parentfound = "Smarter4Ever"}

    return parentfound;
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
  plugin.storage.setItem('configuration', config);
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

// Method to add a fast    
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
		resetAndSave(fasts) ;
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

function onLoaded() {
 //setTimeout(()=>{
 // if (plugin.prefs == undefined) {
 //   window.location.reload()}},2000);
 // if (!plugin) {
 // plugin = new NomiePlugin({
 //   name: pluginname,
 //       emoji: pluginemoji,
 //       avatar: "data:image/jpeg;base64,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",
 //       description: "Intermitted Fasting Timer",
 //       uses: ["createNote", "getLocation","selectTrackables","getTrackable"],
 //       version: "0.5",
 //       addToCaptureMenu: true,
 //       addToMoreMenu: true,
 //       addToWidgets: true,
 //     })
//  }
}

 onMount(async () => {
  loadInitParams();
 })

</script>
<LibLoader url={PlugiAapiUrl}
on:loaded="{onLoaded}" />

{#if mode == "modal"  || mode =="widget"}
<Theme bind:theme />
{#if inNomie}
{#if mode == "modal"}
<Header company={parent} platformName={pluginname} on:click={showMain}>
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
<Info parent={parent} pluginname={pluginname} pluginemoji={pluginemoji} on:exitinfo={showMain}/>
{:else if view == "settings"}
<Settings pluginname={pluginname} pluginemoji={pluginemoji} bind:theme={theme} on:resetall={resetAll} on:savetemplate={saveTemplate} bind:config plugin={plugin} on:savesettings={saveSettings} on:exitsettings={()=>{showMain()}}/>
{:else if view == "mainexercise"}
<MainExercise bind:mode={mode} pluginname={pluginname} pluginemoji={pluginemoji} parent={parent} plugin={plugin} bind:config={config} bind:current={currentexercise} bind:fastname={fastname} bind:fastduration={fastduration} bind:fastdescription={fastdescription} on:exitexercise={showMain} on:storecurrentfast={storeCurrentFast} on:storeprevious={storePrevious} on:logfast={logFast}/>
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
        <h5 style="text-align:center">This is a plugin for {parent}</h5>
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