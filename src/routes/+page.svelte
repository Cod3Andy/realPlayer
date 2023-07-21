<script lang="ts">
    let audio: HTMLAudioElement
    let tracks: File[] = [];
    let songIndex: number = 0;
    let isPlaying: boolean = false;
    let volume: number = 0.5;
    let timeline = 0;
    let duration: number;
    let sliderValue: number = 0;
    $: progress = 0;

    function handleFileInput(event) {
        const files = event.target.files;
        tracks = [...tracks, ...files];
        audio = new Audio(URL.createObjectURL(tracks[songIndex]));
        console.log( tracks, "tracks");
    }
    function getTrack(track) {
        isPlaying = !isPlaying;
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
        songIndex = songIndex--
        if(songIndex < 0) {
            songIndex = tracks.length - 1
        }
        audio.play()
    }
    function nextAudio () {
        songIndex = songIndex++
        if(songIndex > tracks.length - 1) {
            songIndex = 0
        } else {
            songIndex
        }
        audio.play()
        
    }
    function updateProgress() {
        if (audio) {
            audio.currentTime = progress / 100 * audio.duration
            progress = audio.currentTime / audio.duration * 100
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
</script>	
<player class="player">
<img src="img/gramophone.png" alt="" class="gramophone" />
    <div class="music-container {isPlaying? 'play': ''}">
        <div class="music-info">
            <h4 id="title"></h4>
            <!-- <div class="current-time">{format(audio.currentTime)}</div><div class="total-duration">{format(audio.duration)}</div> -->
            <div class="progress-container">
                <div class="progress"></div>
                <!-- <input type="range" class="seek-slider" min={0} max={duration} bind:value={sliderValue} on:input={() => timeChange(sliderValue)}> -->
                <input type="range" class="seek-slider" min={0} max={100} step={1} bind:value={progress} on:change={updateProgress}>
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