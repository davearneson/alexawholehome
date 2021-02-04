---


---

<h1 id="ha-accessible-alexa-multi-room-audio-setup">HA Accessible Alexa Multi Room Audio Setup</h1>
<p><img src="https://i.imgur.com/TF3Gppd.png" alt="Lovelace View for Whole Home Alexa Audio"></p>
<p>This is just a quick project to setup a way to send and monitor audio with Alexa Multi-Room Music Groups via Lovelace in Home Assistant. There is not a streamlined way to do this, and I hate having to do it through the Alexa app. This helps for times that I don’t want to yell long commands, or even if I want to start music to groups from Home Assistant for when I am arriving home or some other automation purpose.</p>
<p><strong><strong>You must create the Alexa Multi-Room Music Groups in the Alexa app.</strong></strong></p>
<h2 id="alexa-media-player-setup">Alexa Media Player Setup</h2>
<p>This project uses the <a href="https://github.com/custom-components/alexa_media_player">Alexa Media Player</a> custom component to interact with my Alexa Devices. I won’t go into detail on setting this up, because I haven’t deviated any from the standard setup for this custom component.</p>
<p><a href="https://github.com/custom-components/alexa_media_player/wiki#play-in-alexa-groups">There is currently a limitation to Alexa Media Player</a> which can be alleviated using this setup. You can start and stop music to individual echo devices, but you can’t send music directly to an Alexa Multi-Room Music Group. The workaround that I’ve found to be most consistent is sending a custom simulated voice command to Alexa through the component, like “Play Jazz on the Everywhere Music Group.”</p>
<h2 id="home-assistant-helpers">Home Assistant Helpers</h2>
<p>I used HA’s helpers to set up an input_select entity and an input_text entity.</p>
<p>I created the input_select entity in the UI by going to Configuration/Helpers and clicking the add button, then “Dropdown.” I then added a name, in my case “Alexa Music Groups”. This entity will be used to let me choose which Alexa Music Group I want to use to play whatever music or audio I choose.</p>
<p>Mine are:</p>
<pre><code>Everywhere
Downstairs
Upstairs
Front of House
Master Bedroom
</code></pre>
<p>I also changed the icon to “mdi:amazon”</p>
<p><img src="https://i.imgur.com/DKnwteO.png" alt="Helper Setup for Whole Home Alexa Audio"></p>
<p>I then created the input_text entity in the UI by going to Configuration/Helpers and clicking the add button, then “Text.” I then added a name, in my case “Alexa Text Input for Groups”. This entity will be used to type text for whatever music or audio I want sent to the Alexa Music Groups. I changed the icon to “mdi:music”</p>
<p><img src="https://i.imgur.com/tMYBSND.png" alt="Helper Setup for Whole Home Alexa Audio"></p>
<h1 id="scripts">Scripts</h1>
<p>I only created two scripts for use in this project. The first one is what I use to start the audio on the group that I have selected. (This is “Play Music on Alexa Groups.”) The other is an (overkill) fail-safe to make sure all music and audio stops if I want it to stop. I made these as scripts so I could control them both through automations and manual input buttons in the UI.</p>
<pre><code>##############################################################################
###Script to send data from helper entities to Alexa via Alexa Media Player###
##############################################################################
alexa_input_music:
  alias: Play Music on Alexa Groups
  sequence:
    - service: media_player.play_media
      data_template:
        entity_id: media_player.echo_link
        media_content_id: "play {{ states('input_text.alexa_text_input_for_groups') }} on the {{ states('input_select.alexa_music_groups') }} group"
        media_content_type: custom
</code></pre>
<p>If the text input is set to “Classical Music” and the input select is set to “Everywhere,” this script will send the simulated voice command, “play Classical Music on the Everywhere group” to my Echo Link when it is run. <strong>Note the need to use “data_template” for this script to work.</strong></p>
<pre><code>##############################################################################        
#################Script to stop all audio on all groups#######################
##############################################################################
stop_alexa_music:
  alias: Stop Music on Alexa Devices
  sequence:
    - service: media_player.play_media
      data:
        entity_id: media_player.echo_link
        media_content_id: stop music on everywhere group
        media_content_type: custom
    - service: media_player.play_media
      data:
        entity_id: media_player.echo_link
        media_content_id: stop music on front of house group
        media_content_type: custom
    - service: media_player.play_media
      data:
        entity_id: media_player.echo_link
        media_content_id: stop music on downstairs group
        media_content_type: custom
    - service: media_player.play_media
      data:
        entity_id: media_player.echo_link
        media_content_id: stop music on upstairs group
        media_content_type: custom
    - service: media_player.play_media
      data:
        entity_id: media_player.echo_link
        media_content_id: stop music on master bedroom group
        media_content_type: custom
    - service: media_player.play_media
      data:
        entity_id: media_player.echo_link
        media_content_id: stop music
        media_content_type: custom
</code></pre>
<p>This one is just sending “stop music” commands to all possible groups.</p>
<h1 id="card-for-lovelace-ui">Card for Lovelace UI</h1>
<p>Next, I combined everything into a simple Lovelace card:</p>
<p><img src="https://i.imgur.com/IzILjYb.png" alt="Custom Card for Lovelace UI"></p>
<pre><code>type: vertical-stack
cards:
  - type: entity
    entity: input_text.alexa_text_input_for_groups
  - type: horizontal-stack
    cards:
      - type: entity
        entity: input_select.alexa_music_groups
      - type: horizontal-stack
        cards:
          - type: button
            tap_action:
              action: toggle
            entity: script.alexa_input_music
            icon: 'mdi:play'
            icon_height: 50px
            name: Play
          - type: button
            tap_action:
              action: toggle
            entity: script.stop_alexa_music
            icon: 'mdi:stop'
            icon_height: 50px
            name: Stop All

</code></pre>
<p>If you click on the Text Input in Lovelace, you can type in the text representing the music or audio you want to send to Alexa:</p>
<p><img src="https://i.imgur.com/Udll4uP.png" alt="Text Input for Lovelace UI"></p>
<p>You can click on the Dropdown entity in Lovelace to choose which Multi-Room Music Group you want to send your request to:</p>
<p><img src="https://i.imgur.com/Zzrd2NT.png" alt="Dropdown Card for Lovelace UI"></p>
<p>Now, all you have to do is click “Play” to launch the script and the music you requested will play on the group you chose! To organize everything, I placed the card in a Lovelace view that contained all of my Alexa Media devices as media cards. This allows me to see everything playing in one place, and gives me controls to pause, play, skip, etc.</p>
<p><img src="https://i.imgur.com/IzILjYb.png" alt="Custom Card for Lovelace UI"></p>

