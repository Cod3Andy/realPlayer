<script lang="ts">
    let audio: HTMLAudioElement
    let tracks: File[] = [];
    let songIndex: number = 0;
    let isPlaying: boolean = false;
    let volume: number = 0.5;
    let timeline = 0;
    let duration: number;
    let totalTimeDisplay = "loading...";
	let currTimeDisplay = "0:00:00";
    let sliderValue: number = 0;
    let ended: boolean = false;
    $: progress = 0;

    function handleFileInput(event) {
        const files = event.target.files;
        tracks = [...tracks, ...files];
        audio = new Audio(URL.createObjectURL(tracks[songIndex]));
        console.log( tracks, "tracks");
    }
    function getTrack(track) {
        audio.pause();
        isPlaying = !isPlaying;
        if(isPlaying === false) {
            isPlaying = true
        }
        audio = new Audio(URL.createObjectURL(track));
        isPlaying? audio.play() : audio.pause();
        audio.play();
    }
    function playPauseAudio() {
        isPlaying = !isPlaying;
        isPlaying? audio.play() : audio.pause();
        console.log(isPlaying);
    }
    function timeChange(sliderValue) {
        timeline = sliderValue
    }
    function timeControl(sliderValue) {
        duration = audio.duration
        audio.currentTime = sliderValue
    };
    function volumeControl() {
        if (audio) {
            audio.volume = volume
        }
    }
    function muteAudio() {
        audio.volume > 0 ? (audio.volume = 0) : (audio.volume = 0.5);
        volume = audio.volume
    }
    function loopAudio () {
    audio.loop === true ? audio.loop = false : audio.loop = true;
    }
    function prevAudio () {
        songIndex--
        if(songIndex < 0) {
            songIndex = tracks.length - 1
        }
        if(isPlaying === false) {
            isPlaying = true
        }
        audio.pause()
        audio = new Audio(URL.createObjectURL(tracks[songIndex]));
        audio.play()
    }
    function nextAudio () {
        songIndex++
        if(songIndex > tracks.length - 1) {
            songIndex = 0
        }
        audio.pause()
        if(isPlaying === false) {
            isPlaying = true
        }
        audio = new Audio(URL.createObjectURL(tracks[songIndex]));
        audio.play()
    }
    function setProgress() {
        if (audio) {
            audio.currentTime = progress / 100 * audio.duration 
        }
    }
	// $:tracks[songIndex].onloadedmetadata = () => {
	// 	duration = audio.duration;
	// 	updateProgress();
	// }

    function updateProgress() {
        progress = audio.currentTime * (100 / duration);
		
		let currHrs = Math.floor((audio.currentTime / 60) / 60);
		let currMins = Math.floor(audio.currentTime / 60);
		let currSecs = Math.floor(audio.currentTime - currMins * 60);
		
		let durHrs = Math.floor( (duration / 60) / 60 );
		let durMins = Math.floor( (duration / 60) % 60 );
		let durSecs =  Math.floor(duration - (durHrs*60*60) - (durMins * 60));
		
		if(currSecs < 10) currSecs = `0${currSecs}`;
		if(durSecs < 10) durSecs = `0${durSecs}`;
		if(currMins < 10) currMins = `0${currMins}`;
		if(durMins < 10) durMins = `0${durMins}`;
		
		currTimeDisplay = `${currHrs}:${currMins}:${currSecs}`;
		totalTimeDisplay = `${durHrs}:${durMins}:${durSecs}`;
		
		if (audio.ended) {
			nextAudio();
		}
    }
    // function format(seconds) {
    //     if (audio) {
    //         const minutes = Math.floor(seconds / 60)
    //         seconds = Math.floor(seconds % 60)
    //             if(seconds < 10) {
    //                 seconds = '0' + seconds
    //             }
    //         return `${minutes}:${seconds}`
    //     } else {
    //         return '00:00'
    //     }
    // }
    function getPercentage(currentTime) {
        const percentage = (Math.floor(currentTime) / duration) * 100 + "%"
        return percentage
    }
    // let currentTime = `${}`
</script>	
<player class="player">
<img src="img/gramophone.png" alt="" class="gramophone" />
    <div class="music-container {isPlaying? 'play': ''}">
        <div class="music-info">
            <p class=" font-mono font-bold text-xl text-primary-100">{isPlaying? tracks[songIndex].name : ''}</p>
            <div class="current-time">{currTimeDisplay}</div><div class="total-duration">{totalTimeDisplay}</div>
            <div class="progress-container">
                <div class="progress"></div>
                <!-- <input type="range" class="seek-slider" min={0} max={duration} bind:value={sliderValue} on:input={() => timeChange(sliderValue)}> -->
                <input type="range" class="seek-slider" min={0} max={100} step={1} bind:value={progress} on:change={setProgress}>
                <seek-slider />
            </div>
        </div>
        <audio src='' id="audio" />
        <div class="img-container">
            <img src="img/vinyl_disk.png" alt="music_cover" id="cover">
        </div>
        <div class="navigation">
            <button id="prev" class="action-btn" on:click={prevAudio}>
                <i class="fas fa-backward"></i>
            </button>
            <button id="play" class="action-btn action-btn-big" on:click={playPauseAudio}>
                <i class="fas {isPlaying? 'fa-pause' : 'fa-play'}"></i>
            </button>
            <button id="next" class="action-btn" on:click={nextAudio}>
                <i class="fas fa-forward"></i>
            </button> 
            <button id="loop" class="action-btn" on:click={loopAudio}>
                <i class="fas fa-arrows-rotate"></i>
            </button>
            <div class="Volume">
                <button id="mute" class="action-btn" on:click={muteAudio}>
                    <i class="fas {volume > 0.7 && volume > 0 ? 'fa-volume-high' : 'fa-volume-low'} {volume === 0 ? 'fa-volume-xmark' : ''}"></i>
                </button>
                <div class="slider" style="width: {volume * 100}%">
                    <input type="range" min={0} max={1} step={0.01} bind:value={volume} on:change={volumeControl}>
                </div>
            </div>
            <label for="upload" class="uploadBtn">Select songs</label>
            <input multiple type="file" class="hidden" id="upload" on:change={handleFileInput} accept="audio/*"/>
        </div>
        <div class="playlist">
            {#each tracks as track }
                <div class="playsongs" on:pointerdown={()=>{getTrack(track)}}>
                    {track.name}
                </div>
            {/each}
        </div>
    </div>
</player> 