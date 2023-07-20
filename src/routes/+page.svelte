<script lang="ts">
    let audio: HTMLAudioElement
    let tracks: File[] = [];
    let currentTrack
    let songIndex: number = 0;
    let isPlaying: boolean = false
    let volume: number = 0.5
    let timeline = 0
    let duration: number
    let sliderValue: number = 0
    $: progress = 0
    $: check = tracks.length

    function handleFileInput(event) {
        const files = event.target.files;
        tracks = [...tracks, ...files];
        console.log( tracks, "tracks");
    }
    function getTrack(track) {
        currentTrack = new Audio()
        currentTrack.src = URL.createObjectURL(track)
        currentTrack.play()
    }
    
        function fileInput(event) {
            const file: HTMLAudioElement = event.target.files;
            audio = new Audio();
            audio.src = URL.createObjectURL(file);
            // tracks.push(file)
            tracks = [...tracks, ...file];
            // console.log(tracks, 'test')
        }
        function playAudio() {
            !isPlaying
            isPlaying? audio.pause() : audio.play()
        }
        function pauseAudio() {
            isPlaying = false;
            audio.pause()
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
            audio.volume > 0 ? (audio.volume = 0) : (audio.volume = 0.5)
        }
        function loopAudio () {
        audio.loop === true ? audio.loop = false : audio.loop = true
        playAudio()
    }
        function prevAudio () {
            songIndex--
            if(songIndex < 0) {
                songIndex = tracks.length - 1
            }
            tracks[songIndex]
            // audio.currentTime = 0
            playAudio()
        }
        function nextAudio () {
            songIndex++
            if(songIndex > tracks.length - 1) {
                songIndex = 0
            }
            tracks[songIndex]
            // audio.currentTime = 0
            playAudio()
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
        <h1 class="z-10 font-mono text-4xl ">Ol' Music Player</h1>
        <img src="img/gramophone.png" alt="" class="gramophone">
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
                    <button id="play" class="action-btn action-btn-big" on:click={playAudio}>
                        <i class="fas {isPlaying? 'fa-pause' : 'fa-play'}"></i>
                    </button>
                    <button id="next" class="action-btn" on:click={nextAudio}>
                        <i class="fas fa-forward"></i>
                    </button> 
                    <button id="loop" class="action-btn" on:click={loopAudio}>
                        <i class="fas fa-arrows-rotate"></i>
                    </button>
                    <button id="mute" class="action-btn" on:click={muteAudio}>
                        <i class="fas fa-volume-mute"></i>
                    </button>
                    <div class="Volume">
                        <div class="slider" style="width: {volume * 100}%">
                            <input type="range" min={0} max={1} step={0.01} bind:value={volume} on:change={volumeControl}>
                            <p>{volume*100}%</p>
                        </div>
                        <input multiple type="file" class="cool_button" on:change={handleFileInput} accept="audio/*"/>
                    </div>
                </div>
            </div>
            <div>
                <h2>Selected files:</h2>
                {#each tracks as track }
                    <div class="bg-gray-500" on:pointerdown={()=>{getTrack(track)}}>
                        {track.name}
                    </div>
                {/each}
            Playlist: {check}
            </div>
    </player> 