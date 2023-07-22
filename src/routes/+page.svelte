<script lang="ts">
    let tracks: File[] = [];
    let songIndex: number = 0;
    let audio: HTMLAudioElement;
    let isPlaying: boolean = false;
    let isLooped: boolean = false;
    let isLiked: boolean = false;
    let volume: number = 0.5;
    let duration: number;
    let totalTimeDisplay = "loading...";
	let currTimeDisplay = "0:00:00";
    let progress: number = 0;

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
            isPlaying = true;
        }
        audio = new Audio(URL.createObjectURL(track));
        isPlaying? audio.play() : audio.pause();
        audio.play();
    }
    function playPauseAudio() {
        audio === null ? null : 
        isPlaying = !isPlaying;
        isPlaying? audio.play() : audio.pause();
    }
    function volumeControl() {
        if (audio) {
            audio.volume = volume;
        }
    }
    function muteAudio() {
        audio.volume > 0 ? (audio.volume = 0) : (audio.volume = 0.5);
        volume = audio.volume;
    }
    function loopAudio () {
        isLooped = !isLooped;
        audio.loop === true ? audio.loop = false : audio.loop = true;
    }
    function prevAudio () {
        songIndex--;
        if(songIndex < 0) {
            songIndex = tracks.length - 1;
        }
        if(!isPlaying) {
            isPlaying = true;
        }
        audio.pause();
        audio = new Audio(URL.createObjectURL(tracks[songIndex]));
        audio.play();
    }
    function nextAudio () {
        songIndex++;
        if(songIndex > tracks.length - 1) {
            songIndex = 0;
        }
        audio.pause()
        if(!isPlaying) {
            isPlaying = true;
        }
        audio = new Audio(URL.createObjectURL(tracks[songIndex]));
        audio.play();
    }
    function setProgress() {
        if (audio) {
            audio.currentTime = progress / 100 * audio.duration;
        }
    }
	$: isPlaying? updateProgress() : null;

    function updateProgress() {
		duration = audio.duration;
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
    function likeAudio () {
        isLiked = !isLiked;
    }
</script>	
<player class="player">
<img src="img/gramophone.png" alt="" class="gramophone" />
    <div class="music-container {isPlaying? 'play': ''}">
        <div class="music-info">
            <p class=" font-mono font-bold text-xl text-primary-100" id="songTitle">{isPlaying? tracks[songIndex].name  : ''}</p>
            <div class="flex justify-between">
                <span id="progress-time">{currTimeDisplay}</span>
                <span id="track-duration">{totalTimeDisplay}</span>
            </div>
            <div class="progress-container">
                <div class="progress"></div>
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
                <i class="fas {isLooped === true ? 'fa-arrows-rotate text-red-600 scale-150 ' : 'fa-arrows-rotate'}"></i>
            </button>
            <div class="Volume group">
                <button id="mute" class="action-btn" on:click={muteAudio}>
                    <i class="fas {volume > 0.7 && volume > 0 ? 'fa-volume-high' : 'fa-volume-low'} {volume === 0 ? 'fa-volume-xmark' : ''}"></i>
                </button>
                <div class="slider hidden group-hover:block" style="width: {volume * 100}%">
                    <input type="range" min={0} max={1} step={0.01} bind:value={volume} on:change={volumeControl}>
                </div>
            </div>
            <label for="upload" class="uploadBtn">Select songs</label>
            <input multiple type="file" class="hidden" id="upload" on:change={handleFileInput} accept="audio/*"/>
        </div>
        <div class="playlist">
            {#each tracks as track }
                <div class="playlist-elements">
                    <div class="playsongs" on:pointerdown={()=>{getTrack(track)}}>
                        {track.name}
                    </div>
                    <p></p>
                    <button class="fa-{isLiked? 'solid' : 'regular'} fa-heart like-btn {isLiked? 'text-4xl' : 'text-3xl'}" on:click={likeAudio}></button>
                </div>
            {/each}
        </div>
    </div>
</player> 