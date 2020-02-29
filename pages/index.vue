<template>
  <div class="container">
    rendering
    <video
      id="video"
      autoPlay
      playsInline
    />
  </div>
</template>

<script>
import Logo from '~/components/Logo.vue'
import Janus from '~/components/janus'



export default {
  components: {
    Logo
  },
  mounted() {
    const server = "your server"
    const opaqueId = "streamingtest-"+Janus.randomString(12);
    let streaming = null;

    Janus.init({debug: "all", callback: function() {
			let janus = new Janus(
				{
					server: server,
					success: function() {
						janus.attach(
							{
								plugin: "janus.plugin.streaming",
								opaqueId: opaqueId,
								success: function(pluginHandle) {
                   const body = { "request": "watch", id: parseInt(1) };
                   streaming = pluginHandle
                   streaming.send({"message": body});
								},
								error: function(error) {
									console.log("Error attaching plugin... " + error);
								},
								onmessage: function(msg, jsep) {
									const result = msg["result"];
									console.log(msg)
									if(jsep !== undefined && jsep !== null) {
										streaming.createAnswer(
											{
												jsep: jsep,
												media: { audioSend: false, videoSend: false, data: true },
												success: function(jsep) {
													var body = { "request": "start" };
													streaming.send({"message": body, "jsep": jsep});
												},
												error: function(error) {
													console.log("WebRTC error... " + JSON.stringify(error));
												}
											});
									}
								},
								onremotestream: function(stream) {
									var addButtons = false;
										addButtons = true
											var videoTracks = stream.getVideoTracks();
											if(videoTracks === null || videoTracks === undefined || videoTracks.length === 0)
                        return;
                        
                  let video = document.getElementById('video')
                  // video.srcObject = stream;// webinar or p2p
                  // console.log("rendering", stream)
                  Janus.attachMediaStream(video, stream);
                  
								}
							});
					},
					error: function(error) {
						bootbox.alert(error, function() {
							window.location.reload();
						});
					}
        });
        
    }})
  }
}
</script>

<style>
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.title {
  font-family: 'Quicksand', 'Source Sans Pro', -apple-system, BlinkMacSystemFont,
    'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}
</style>
