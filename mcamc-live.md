---
layout: default
title: MCAMC Live Round
permalink: /mcamc/live/
meta: | 
  <link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
  <script type="text/x-mathjax-config">
    MathJax.Hub.Config({
      tex2jax: {
        skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
        inlineMath: [['$','$']]
      }
    });
  </script>
  <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
---
<button id="authorize_button" style="display: none;">Authorize</button>
<button id="signout_button" style="display: none;">Sign Out</button>
<div class="cwrapper">
<div id="live-table" class="mcamc-table" style="float: left"></div>
<div style="width: 2px"></div>
<div id="live-table2" class="mcamc-table" style="float: left"></div>
<div style="width: 2px"></div>
<div id="live-table3" class="mcamc-table" style="float: left"></div>

<script type="text/javascript">
      // Client ID and API key from the Developer Console
      var CLIENT_ID = '59368639887-aovrnh1v37ceo7fk3jlu93tkjociueb9.apps.googleusercontent.com';
      var API_KEY = 'AIzaSyAPAx4gDVIjvOCrid7f4lWv2N36EPZBG3U';

      // Array of API discovery doc URLs for APIs used by the quickstart
      var DISCOVERY_DOCS = ["https://sheets.googleapis.com/$discovery/rest?version=v4"];

      // Authorization scopes required by the API; multiple scopes can be
      // included, separated by spaces.
      var SCOPES = "https://www.googleapis.com/auth/spreadsheets.readonly";

      var authorizeButton = document.getElementById('authorize_button');
      var signoutButton = document.getElementById('signout_button');

      /**
       *  On load, called to load the auth2 library and API client library.
       */
      function handleClientLoad() {
        gapi.load('client:auth2', initClient);
      }

      /**
       *  Initializes the API client library and sets up sign-in state
       *  listeners.
       */
      function initClient() {
        gapi.client.init({
          apiKey: API_KEY,
          clientId: CLIENT_ID,
          discoveryDocs: DISCOVERY_DOCS,
          scope: SCOPES
        }).then(function () {
          // Listen for sign-in state changes.
          gapi.auth2.getAuthInstance().isSignedIn.listen(updateSigninStatus);

          // Handle the initial sign-in state.
          updateSigninStatus(gapi.auth2.getAuthInstance().isSignedIn.get());
          authorizeButton.onclick = handleAuthClick;
          signoutButton.onclick = handleSignoutClick;
        }, function(error) {
          appendPre(JSON.stringify(error, null, 2));
        });
      }

      /**
       *  Called when the signed in status changes, to update the UI
       *  appropriately. After a sign-in, the API is called.
       */
      function updateSigninStatus(isSignedIn) {
        if (isSignedIn) {
          authorizeButton.style.display = 'none';
          signoutButton.style.display = 'none';
          listMajors();
        } else {
          authorizeButton.style.display = 'block';
          signoutButton.style.display = 'none';
        }
      }

      /**
       *  Sign in the user upon button click.
       */
      function handleAuthClick(event) {
        gapi.auth2.getAuthInstance().signIn();
      }

      /**
       *  Sign out the user upon button click.
       */
      function handleSignoutClick(event) {
        gapi.auth2.getAuthInstance().signOut();
      }

      /**
       * Append a pre element to the body containing the given message
       * as its text node. Used to display the results of the API call.
       *
       * @param {string} message Text to be placed in pre element.
       */
      function appendPre(message) {
        var pre = document.getElementById('content');
        var textContent = document.createTextNode(message + '\n');
        pre.appendChild(textContent);
      }

      function analyzeRow(row) {
      	var rowData = {};
      	sum = row.slice(2, row.length).reduce((a, b) => parseInt(a) + parseInt(b));
      	rowData.score = sum % 1000;
      	rowData.setsComplete = Math.floor(sum / 1000);
      	rowData.teamName = row[1];
      	rowData.teamNumber = row[0];
      	return rowData
      }

      

      var scores = [];

      function listMajors() {
        gapi.client.sheets.spreadsheets.values.get({
          spreadsheetId: '17oX1WsQa5oSJfoEinkW8ZTIwkPDkF5mQI_s3LevkeLc',
          range: 'Data!2:36',
        }).then(function(response) {
          var range = response.result;
          if (range.values.length > 0) {
            for (i = 0; i < range.values.length; i++) {
              var row = range.values[i];
              // Print columns A and E, which correspond to indices 0 and 4.
              rowData = analyzeRow(row);
              scores[i] = [];
              scores[i][0] = rowData.teamNumber;
              scores[i][1] = rowData.teamName;
              scores[i][2] = rowData.score;
              scores[i][3] = rowData.setsComplete;
            }
          }
        }, function(response) {
          
        });

        
        scores.sort(function(a,b) {
           return b[2] - a[2]
        });

       var html = "<table><tbody><tr><td>#</td><td>Name         </td><td>Score</td><td>Sets</td></tr>";
       
       var split = Math.round((scores.length/3));
    
       for (var i = 0; i < split; i++) {
        html+="<tr>";
        html+="<td>"+scores[i][0]+"</td>";
        html+="<td>"+scores[i][1]+""+"</td>";
        html+="<td style=\"text-align:right\">"+scores[i][2]+"</td>";
        html+="<td>"+scores[i][3]+"/8"+"</td>";
        html+="</tr>";
       }
        html+="</tbody></table>";

       var html2 = "<table><tbody><tr><td>#</td><td>Name         </td><td>Score</td><td>Sets</td></tr>";
    
       for (var i = split; i < (split*2); i++) {
        html2+="<tr>";
        html2+="<td style=\"border-left: solid 1px black\">"+scores[i][0]+"</td>";
        html2+="<td>"+scores[i][1]+""+"</td>";
        html2+="<td style=\"text-align:right\">"+scores[i][2]+"</td>";
        html2+="<td>"+scores[i][3]+"/8"+"</td>";
        html2+="</tr>";
       }
        html2+="</tbody></table>";
        
       var html3 = "<table><tbody><tr><td>#</td><td>Name         </td><td>Score</td><td>Sets</td></tr>";
    
       for (var i = split*2; i < scores.length; i++) {
        html3+="<tr>";
        html3+="<td style=\"border-left: solid 1px black\">"+scores[i][0]+"</td>";
        html3+="<td>"+scores[i][1]+""+"</td>";
        html3+="<td style=\"text-align:right\">"+scores[i][2]+"</td>";
        html3+="<td>"+scores[i][3]+"/8"+"</td>";
        html3+="</tr>";
       }
        html3+="</tbody></table>";

        document.getElementById("live-table").innerHTML = html;
        document.getElementById("live-table2").innerHTML = html2;
        document.getElementById("live-table3").innerHTML = html3;
        setTimeout(listMajors, 5000);
      }
</script>
</div>

<script async defer src="https://apis.google.com/js/api.js"
      onload="this.onload=function(){};handleClientLoad()"
      onreadystatechange="if (this.readyState === 'complete') this.onload()">
</script>


