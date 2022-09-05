<script>
    import { createEventDispatcher } from 'svelte';
    import moment from 'moment';
    import {
      Button,
      Content,
        Grid,
        Row,
        Column,
    } from "carbon-components-svelte";
    import { saveToLS, readFromLS } from '../storage/storage';
   
    export let pluginname;
    export let pluginemoji;

   export let fastname ="default";
   export let fastduration = 1;
   export let fastdescription = "default description"
   export let current;
   export let plugin;
   export let config;
   export let mode;
   
   let status = "About to start Fasting";
   if(current.fasting){
            status = "You are currently Fasting";
            fastname = current.name;
            fastduration = current.duration;
            fastdescription = current.description;
   }
   var isfasting = JSON.parse(readFromLS('currentfast')) || {description: "",name:"",minutes: "", hours: "", day: "", duration: "", endtime: "", endday: "", fasting: false};
  
   
   if(mode=="widget"){
   setInterval(() =>{
    isfasting = JSON.parse(readFromLS('currentfast')) || {description: "",name:"",minutes: "", hours: "", day: "", duration: "", endtime: "", endday: "", fasting: false};
    start=isfasting},5000)}

   let buttonview = "Start";
   
   var percenttext = "0%";
   var digitsdisplay = "00:00:00";
   var elapsedtime = fastduration.toString();


   
   const dispatch = createEventDispatcher();

   var widgetbgcolor = "";

   $: if(plugin.prefs.theme){
   if (plugin.prefs.theme == "dark"){
     widgetbgcolor = "black";}
   else {widgetbgcolor = "white"}  

   }


   function exitMain(){
        dispatch("exitinfo");
        open = false;
    }

    // FAST SPECIFIC CODE
    var start = {description: "", name:"", minutes: "", hours: "", day: "", duration: "", endtime: "", endday: "", fasting: false};
    var currentFast = current;
    var circleClass = "progress-circle p1";

    init();

    $: if(currentFast){
        if(currentFast.fasting){
            status = "You are currently Fasting"
        }
        else {status = "About to start Fasting"}
    }

    function renderStart(){
};

function init() {
    // check if local storage has been used else get data from local storage
    if(!currentFast.fasting){
        console.log("nothing in storage");
    } else {
        start = current;
        console.log("get start"+start);
        buttonview="End";
    }
    // Render buttons
    renderStart();
};

function storeCurrrentFast() {
    // store timeblock objects in local storage
    saveToLS('currentfast', start);
    dispatch("storecurrentfast",start)
    
};

function buttonClicked() {
    if(buttonview == "Start") {
        buttonview = "End"
        startFast();
        status = "You are currently Fasting"
    }
    else {
        buttonview = "Start"
        endFast();
        status = "About to start Fasting"
    }
}

// TIMER

// get value of fasting duration
start.duration = fastduration.toString();

// start fast
function startFast() {
            // get value of fasting exercise
            start.duration = fastduration;
            start.description = fastdescription;
            start.name = fastname;
            start.hours = moment().format('HH:mm:ss');
            start.minutes = moment().format('mm');
            start.day = moment().calendar();
            start.endtime = moment().add(start.duration, 'hours').format('HH:mm');
            start.endday = moment().add(start.duration, 'hours').calendar();
            start.fasting = true;
            // show end fast button
            buttonview = "End";
            // store in local storage
            storeCurrrentFast();
            renderStart();
    }; 

    function storePrevious(fastended) {
        dispatch("storeprevious",fastended)
    }

    var refreshTimer;
    var percent;
    var starttime;
    var endtime;
    var elapsedtime;
    

    refreshTimer = setInterval(function(){
        starttime = start.day;
        endtime = start.endday;
        var timeDisplay = moment().subtract(start.hours).format('HH:mm:ss');
        var minutes = moment().subtract(start.minutes, 'minutes').format('mm');
        minutes = parseInt(minutes)*1.67;
        var elapsedTime = parseInt(start.duration) - parseInt(timeDisplay);
        var hours = parseInt(timeDisplay)*100;
        percent = Math.round((hours+minutes+1)/(parseInt(start.duration)*100)*100);
        if (start.fasting==true){
        digitsdisplay = timeDisplay;
        elapsedtime=elapsedTime;
        showPercent();
        }
        
    }, 500);

    // function to show percent / circle progress
    function showPercent(){
        percenttext = percent+"%";
        if (percent > 50){
            circleClass = "progress-circle over50 p"+(percent.toString())
        }
        else {circleClass = "progress-circle p"+(percent.toString())}
    };
    var finaltime;
    async function endFast(){
        var fastEnded = moment().subtract(start.hours).format('HH:mm:ss');
        finaltime= fastEnded;
        console.log("end fast");
        // store fast
        if(config.trackers[0] != "none" || config.trackers[1] != "none") {
        var res = {};
        res = await plugin.confirm('Save Results?', 'Concratulations, you finished your fasting period. Would you like to log your results in Nomie?');
         if (res.value) {
            var res2 ={}
            var mood = 0;
            res2 = await plugin.prompt('Mood', 'How was the mood during this fasting? Please provide a number between 1 and 10');
            if (res2.value) {mood = res2.value;}
            if(config.trackeroverrule){
  
    var res3 = {};
    res3 = await plugin.confirm('Log session to Nomie', 'This will log your session in Nomie as you configured via settings');
   // res.value = confirm("Log session");
      if (res3.value) {
        let trackinglog ="";
          if (config.trackers[1] != "None") {
            let timertrack = "#"+config.trackers[1]+"("+finaltime+")";
            trackinglog = timertrack+"\n";
          }
          if (config.trackers[0] != "None") {
          let moodtrack = "#"+config.trackers[0]+"("+(mood).toString()+")";
          trackinglog = trackinglog + moodtrack;
          }
        let logentry = config.logentry;
        logentry = logentry.replace("<duration>", finaltime.toString());
        logentry = logentry.replace("<fast>", current.name);
        const note = "Nomie Fast ⏲:\n"+logentry+"\n\n"+trackinglog;
        dispatch("logfast",note);
        }
      }
    
        // reset variables 
        start = {name:"",description:"",minutes: "", hours: "", day: "", duration: "", endtime: "", endday: "", fasting: false};
        storeCurrrentFast();
        storePrevious(fastEnded);
        // reset display
        digitsdisplay= "00:00:00";
        elapsedtime="";
        percent = 0;
        percenttext = percent+"%";
        // show start button
        buttonview = "Start";
        dispatch("exitexercise");
    }
    else {
        console.log("User does not want to save")
        start = {name:"",description:"",minutes: "", hours: "", day: "", duration: "", endtime: "", endday: "", fasting: false};
        storeCurrrentFast();
        // reset display
        digitsdisplay= "00:00:00";
        elapsedtime="";
        percent = 0;
        percenttext = percent+"%";
        // show start button
        buttonview = "Start";
        dispatch("exitexercise");

    }} 
    else {
        console.log("Nothing will be saved as settings are not configured")
        plugin.alert("Concratulation!", "Concratulations, you finalised a fasting. You do not have any settings on how to log this in Nomie so nothing will be logged");
        start = {name:"",description:"",minutes: "", hours: "", day: "", duration: "", endtime: "", endday: "", fasting: false};
        storeCurrrentFast();
        // reset display
        digitsdisplay= "00:00:00";
        elapsedtime="";
        percent = 0;
        percenttext = percent+"%";
        // show start button
        buttonview = "Start";
        dispatch("exitexercise");
    }
     }    
    


</script>
{#if mode == "modal"}
<Content>
    <Grid>
      <Row>
        <Column>
          <h1 style="text-align:center">{pluginemoji}</h1>
          <h2 style="text-align:center">{pluginname}</h2>
          <h5 style="text-align:center">Fast Exercise</h5>
          <hr>
        </Column>
      </Row>
      <Row>
        <Column>
          <br>
          <h5 style="text-align:center">{fastname}</h5>
        </Column>
      </Row>
      <Row>
        <Column>
          <h5 style="text-align:center">{status}</h5>
        </Column>
      </Row>
      <Row>
        <Column>
          <div class="circle-timer-container">
            <!-- progress circle -->
                <div id="circle" class="{circleClass}">
                    <div id="circle-text">
                        <div id="elapsed-div">
                            <p>Hours to go: </p>
                            <p id="elapsed-time">{elapsedtime}</p>
                        </div>
                        <span id="percent">{percenttext}</span>
                            <div id="timer-display">
                                <h1 id="digits-display">{digitsdisplay}</h1>
                            </div>
                    </div>
                    <div class="left-half-clipper">
                      <div class="first50-bar"></div>
                      <div class="value-bar"></div>
                    </div>
                  </div>    
        </Column>
      </Row>
      <Row>
        <Column><p style="text-align:left;font-size: 0.875em;">Fasting Started:</p></Column>
        <Column><p style="text-align:right;font-size: 0.875em;">Fast End:</p></Column>
      </Row>
      <Row>
        <Column><p style="text-align:left;font-size: 0.875em;">{starttime}</p></Column>
        <Column><p style="text-align:right;font-size: 0.875em;">{endtime}</p></Column>
      </Row>
      <Row>
        <Column>
            <br>
             <span><Button style="width:100%" kind="secondary" on:click={()=>{dispatch("exitexercise")}}>Back</Button></span>
             <br>
        </Column>
      <Column>
          <br>
           <span><Button style="width:100%" on:click={buttonClicked}>{buttonview}</Button></span>
           <br>
      </Column>
  </Row>    
    </Grid>
  </Content>
  {:else if mode == "widget"}
  <div class="widget" style="background-color:{widgetbgcolor}">
  {#if isfasting.fasting}  
   
  <div class="circle-timer-container">
    <!-- progress circle -->
        <div id="circle" class="{circleClass}"  style="transform:scale(0.50);margin-top:-75px;">
            <div id="circle-text">
                <div id="elapsed-div">
                    <p>Hours to go: </p>
                    <p id="elapsed-time">{elapsedtime}</p>
                </div>
                <span id="percent">{percenttext}</span>
                    <div id="timer-display">
                        <h1 id="digits-display">{digitsdisplay}</h1>
                    </div>
            </div>
            <div class="left-half-clipper">
              <div class="first50-bar"></div>
              <div class="value-bar"></div>
            </div>
          </div>  
          </div>
 {:else}
  <br>
  <br>
  <h1 style="text-align:center;line-height:40px">YOU'RE NOT FASTING</h1>
  {/if}
</div>
  {/if}




  <style>
    h2 {
       margin: 0;
       padding: 0;
       font-size: 2.5em;
       font-weight: 400;
       
   }

   .widget{
    width:100%;
    height:100%;
    margin-top: 0;
   }

   .progress-circle {
   font-size: 60px;
   margin: 20px;
   position: relative; /* so that children can be absolutely positioned */
   padding: 0;
   width: 5em;
   height: 5em;
   background-color: #b8c0ee; 
   border-radius: 50%;
   line-height: 5em;
   transform: scale(0.9);
}

.progress-circle:after{
    border: none;
    position: absolute;
    top: 0.35em;
    left: 0.35em;
    text-align: center;
    display: block;
    border-radius: 50%;
    width: 4.3em;
    height: 4.3em;
    background-color: white;
    content: " ";
}
/* Text inside the control */
.progress-circle #circle-text {
    position: absolute;
    line-height: 1em;
    width: 5em;
    text-align: center;
    display: block;
    color: #1f65b6;
    z-index: 2;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
}
.left-half-clipper { 
   /* a round circle */
   border-radius: 50%;
   width: 5em;
   height: 5em;
   position: absolute; /* needed for clipping */
   clip: rect(0, 5em, 5em, 2.5em); /* clips the whole left half*/ 
}
/* when p>50, don't clip left half*/
.progress-circle.over50 .left-half-clipper {
   clip: rect(auto,auto,auto,auto);
}
.value-bar {
   /*This is an overlayed square, that is made round with the border radius,
   then it is cut to display only the left half, then rotated clockwise
   to escape the outer clipping path.*/ 
   position: absolute; /*needed for clipping*/
   clip: rect(0, 2.5em, 5em, 0);
   width: 5em;
   height: 5em;
   border-radius: 50%;
   border: 0.45em solid #1f65b6; /*The border is 0.35 but making it larger removes visual artifacts */
   /*background-color: #4D642D;*/ /* for debug */
   box-sizing: border-box;
  
}
/* Progress bar filling the whole right half for values above 50% */
.progress-circle.over50 .first50-bar {
   /*Progress bar for the first 50%, filling the whole right half*/
   position: absolute; /*needed for clipping*/
   clip: rect(0, 5em, 5em, 2.5em);
   background-color: #1f65b6;
   border-radius: 50%;
   width: 5em;
   height: 5em;
}
.progress-circle:not(.over50) .first50-bar{ display: none; }


/* Progress bar rotation position */
.progress-circle.p0 .value-bar { display: none; }
.progress-circle.p1 .value-bar { transform: rotate(4deg); }
.progress-circle.p2 .value-bar { transform: rotate(7deg); }
.progress-circle.p3 .value-bar { transform: rotate(11deg); }
.progress-circle.p4 .value-bar { transform: rotate(14deg); }
.progress-circle.p5 .value-bar { transform: rotate(18deg); }
.progress-circle.p6 .value-bar { transform: rotate(22deg); }
.progress-circle.p7 .value-bar { transform: rotate(25deg); }
.progress-circle.p8 .value-bar { transform: rotate(29deg); }
.progress-circle.p9 .value-bar { transform: rotate(32deg); }
.progress-circle.p10 .value-bar { transform: rotate(36deg); }
.progress-circle.p11 .value-bar { transform: rotate(40deg); }
.progress-circle.p12 .value-bar { transform: rotate(43deg); }
.progress-circle.p13 .value-bar { transform: rotate(47deg); }
.progress-circle.p14 .value-bar { transform: rotate(50deg); }
.progress-circle.p15 .value-bar { transform: rotate(54deg); }
.progress-circle.p16 .value-bar { transform: rotate(58deg); }
.progress-circle.p17 .value-bar { transform: rotate(61deg); }
.progress-circle.p18 .value-bar { transform: rotate(65deg); }
.progress-circle.p19 .value-bar { transform: rotate(68deg); }
.progress-circle.p20 .value-bar { transform: rotate(72deg); }
.progress-circle.p21 .value-bar { transform: rotate(76deg); }
.progress-circle.p22 .value-bar { transform: rotate(79deg); }
.progress-circle.p23 .value-bar { transform: rotate(83deg); }
.progress-circle.p24 .value-bar { transform: rotate(86deg); }
.progress-circle.p25 .value-bar { transform: rotate(90deg); }
.progress-circle.p26 .value-bar { transform: rotate(94deg); }
.progress-circle.p27 .value-bar { transform: rotate(97deg); }
.progress-circle.p28 .value-bar { transform: rotate(101deg); }
.progress-circle.p29 .value-bar { transform: rotate(104deg); }
.progress-circle.p30 .value-bar { transform: rotate(108deg); }
.progress-circle.p31 .value-bar { transform: rotate(112deg); }
.progress-circle.p32 .value-bar { transform: rotate(115deg); }
.progress-circle.p33 .value-bar { transform: rotate(119deg); }
.progress-circle.p34 .value-bar { transform: rotate(122deg); }
.progress-circle.p35 .value-bar { transform: rotate(126deg); }
.progress-circle.p36 .value-bar { transform: rotate(130deg); }
.progress-circle.p37 .value-bar { transform: rotate(133deg); }
.progress-circle.p38 .value-bar { transform: rotate(137deg); }
.progress-circle.p39 .value-bar { transform: rotate(140deg); }
.progress-circle.p40 .value-bar { transform: rotate(144deg); }
.progress-circle.p41 .value-bar { transform: rotate(148deg); }
.progress-circle.p42 .value-bar { transform: rotate(151deg); }
.progress-circle.p43 .value-bar { transform: rotate(155deg); }
.progress-circle.p44 .value-bar { transform: rotate(158deg); }
.progress-circle.p45 .value-bar { transform: rotate(162deg); }
.progress-circle.p46 .value-bar { transform: rotate(166deg); }
.progress-circle.p47 .value-bar { transform: rotate(169deg); }
.progress-circle.p48 .value-bar { transform: rotate(173deg); }
.progress-circle.p49 .value-bar { transform: rotate(176deg); }
.progress-circle.p50 .value-bar { transform: rotate(180deg); }
.progress-circle.p51 .value-bar { transform: rotate(184deg); }
.progress-circle.p52 .value-bar { transform: rotate(187deg); }
.progress-circle.p53 .value-bar { transform: rotate(191deg); }
.progress-circle.p54 .value-bar { transform: rotate(194deg); }
.progress-circle.p55 .value-bar { transform: rotate(198deg); }
.progress-circle.p56 .value-bar { transform: rotate(202deg); }
.progress-circle.p57 .value-bar { transform: rotate(205deg); }
.progress-circle.p58 .value-bar { transform: rotate(209deg); }
.progress-circle.p59 .value-bar { transform: rotate(212deg); }
.progress-circle.p60 .value-bar { transform: rotate(216deg); }
.progress-circle.p61 .value-bar { transform: rotate(220deg); }
.progress-circle.p62 .value-bar { transform: rotate(223deg); }
.progress-circle.p63 .value-bar { transform: rotate(227deg); }
.progress-circle.p64 .value-bar { transform: rotate(230deg); }
.progress-circle.p65 .value-bar { transform: rotate(234deg); }
.progress-circle.p66 .value-bar { transform: rotate(238deg); }
.progress-circle.p67 .value-bar { transform: rotate(241deg); }
.progress-circle.p68 .value-bar { transform: rotate(245deg); }
.progress-circle.p69 .value-bar { transform: rotate(248deg); }
.progress-circle.p70 .value-bar { transform: rotate(252deg); }
.progress-circle.p71 .value-bar { transform: rotate(256deg); }
.progress-circle.p72 .value-bar { transform: rotate(259deg); }
.progress-circle.p73 .value-bar { transform: rotate(263deg); }
.progress-circle.p74 .value-bar { transform: rotate(266deg); }
.progress-circle.p75 .value-bar { transform: rotate(270deg); }
.progress-circle.p76 .value-bar { transform: rotate(274deg); }
.progress-circle.p77 .value-bar { transform: rotate(277deg); }
.progress-circle.p78 .value-bar { transform: rotate(281deg); }
.progress-circle.p79 .value-bar { transform: rotate(284deg); }
.progress-circle.p80 .value-bar { transform: rotate(288deg); }
.progress-circle.p81 .value-bar { transform: rotate(292deg); }
.progress-circle.p82 .value-bar { transform: rotate(295deg); }
.progress-circle.p83 .value-bar { transform: rotate(299deg); }
.progress-circle.p84 .value-bar { transform: rotate(302deg); }
.progress-circle.p85 .value-bar { transform: rotate(306deg); }
.progress-circle.p86 .value-bar { transform: rotate(310deg); }
.progress-circle.p87 .value-bar { transform: rotate(313deg); }
.progress-circle.p88 .value-bar { transform: rotate(317deg); }
.progress-circle.p89 .value-bar { transform: rotate(320deg); }
.progress-circle.p90 .value-bar { transform: rotate(324deg); }
.progress-circle.p91 .value-bar { transform: rotate(328deg); }
.progress-circle.p92 .value-bar { transform: rotate(331deg); }
.progress-circle.p93 .value-bar { transform: rotate(335deg); }
.progress-circle.p94 .value-bar { transform: rotate(338deg); }
.progress-circle.p95 .value-bar { transform: rotate(342deg); }
.progress-circle.p96 .value-bar { transform: rotate(346deg); }
.progress-circle.p97 .value-bar { transform: rotate(349deg); }
.progress-circle.p98 .value-bar { transform: rotate(353deg); }
.progress-circle.p99 .value-bar { transform: rotate(356deg); }
.progress-circle.p100 .value-bar { transform: rotate(360deg); }

* {
    padding: 0;
    margin: 0;
    font-family: 'Roboto Condensed', sans-serif;
}

div.bar {
        width: 25px;
        height: 4px;
        background-color: #9E9E9E;
        margin: 5px 0;
        border-radius: 2px;
}

div.bar {
    background-color: #cccccc;
}

.line {
    border-top: 1px solid #1f65b6;
    width: 200px;
}
/* main */

.fast-type {
    padding-left: 15px;
}

.select-text {
    color: #9E9E9E;
    font-weight: 300;
    margin-top: 10px;
}
/* timer display */
.circle-timer-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    overflow: hidden;
}
#digits-display {
    color: #4e4e4e;
    font-size: 3rem;
}
#elapsed-div {
    color: #9E9E9E;
    display: flex;
    justify-content: center;
    font-size: 1rem;
}
#elapsed-time {
    margin-left: 5px;
}
/* start and finish times */

.edit-time-container {
    display: flex;
    justify-content: center;
}
.start-finish {
    color: #9E9E9E;
}
/* button */

.graph-label {
    font-size: 0.8rem;
    color: #9E9E9E;
    margin-bottom: 8px;
}

.graph-column {
    display: flex;
    flex-direction: column;
    align-items: center;
}
.graph-bar {
    width: 16px;
}
.graph-eat {
    background-image: linear-gradient(#ffffff , #E2E2E2);
}
.graph-fast {
    background-image: linear-gradient(#1f65b6 , #ffffff);
}



.firstPara {
    justify-content: center;
    color: #4e4e4e;
}

.column {
    justify-content: center;
}
/* end fast time screen */

 </style>

