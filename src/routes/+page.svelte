<script lang="ts">
    let tracks: File[] = [];
    let likedtracks: File[] = [];
    let songIndex: number = 0;
    let audio: HTMLAudioElement;
    let isLoaded: boolean = false;
    let isLooped: boolean = false;
    let isLiked: boolean = false;
    let isMuted: boolean = false;
    let isPlaying: boolean = false;
    let volume: number = 0.5;
    let duration: number;
    let totalTimeDisplay = "";
	let currTimeDisplay = "";
    let progress: number = 0;
    let searchText: string = '';
    let filteredPlaylist = tracks.filter(filterTracks);
    let form = {
    searchString: ''
    };
    let searchString = '';
    $: isPlaying? updateProgress() : null;
    $: tracks = tracks.filter((track) => {
    return track.name.toLowerCase().includes(searchString.toLowerCase());
  });

    const submitSearch = (e: Event) => {
    searchString = form.searchString;
  };

    function handleFileInput(event: { target: { files: any; }; }){
        isLoaded = !isLoaded;
        const files = event.target.files;
        tracks = [...tracks, ...files];
        audio = new Audio(URL.createObjectURL(tracks[songIndex]));
    };

    function getTrack(track: Blob | MediaSource){
        audio.pause();
        audio = new Audio(URL.createObjectURL(track));
    };

    function playPauseAudio(){
        isPlaying = !isPlaying;
        isPlaying ? audio.play() : audio.pause();
    };

    function loopAudio(){
        isLooped = !isLooped;
        isLooped ? audio.loop = true : audio.loop = false;
    };
    
    function muteAudio(){
        isMuted = !isMuted;
        isMuted ? audio.muted = true : audio.muted = false;
        isMuted ? volume = 0 : volume = 0.5;
    };

    function prevAudio(){
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
    };

    function nextAudio(){
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
    };

    function setProgress(){
        if (audio) {
            audio.currentTime = progress / 100 * audio.duration;
        }
    };
    
    function updateProgress(){
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
		
		if (audio.ended) {
            nextAudio();
		}
        currTimeDisplay = `${currHrs}:${currMins}:${currSecs}`;
        totalTimeDisplay = `${durHrs}:${durMins}:${durSecs}`;
    };
    
    function filterTracks(track){
        return track.name.toLowerCase().includes(searchText.toLowerCase());
    };

    function likeAudio(track: File){
        isLiked = !isLiked;
        likedtracks = tracks.push(track);
    };

    function deleteTrack(track: File){
        tracks = tracks.filter(t => t !== track);
    };
</script>	
<player class="flex overflow-hidden">
<div class="w-full h-full absolute left-0 top-0 z-0 bg-gray-950" />
    <navigation class="flex justify-between absolute bottom-0 w-full z-[1] mb-4 px-1">
        <button on:click={prevAudio} class="action-btn">
            <i class="fas fa-backward"></i>
        </button>
        <button on:click={playPauseAudio} class="action-btn action-btn-big">
            <i class="fas {isPlaying? 'fa-pause' : 'fa-play'}"></i>
        </button>
        <button on:click={nextAudio} class="action-btn">
            <i class="fas fa-forward"></i>
        </button>
        <div class="w-2/3">
            <div class="flex flex-col">
                <span class=" max-w-xl font-mono font-bold text-xl text-red-600">{isPlaying? tracks[songIndex].name : ''}</span>
                <div class="flex justify-between text-white font-mono font-bold" >
                    <span>{currTimeDisplay}</span> <span>{totalTimeDisplay}</span>
                </div>
            </div> 
            <div class="relative bg-white rounded-md cursor-pointer my-3 mx-0 h-2 w-full">
                <div class="progress" style="width: {progress}%"></div>
                <input type="range" class="seek-slider" min={0} max={100} step={1} bind:value={progress} on:input={setProgress} on:timeupdate={() => updateProgress()}>
            </div>
        </div>
        <button on:click={loopAudio} class="action-btn">
            <i class="fas {isLooped === true ? 'fa-arrows-rotate text-red-600 scale-150 ' : 'fa-arrows-rotate'}"></i>
        </button>
        <div class="Volume ">
            <button on:click={muteAudio} class="action-btn">
                <i class="fas {volume > 0.7 && volume > 0 ? 'fa-volume-high' : 'fa-volume-low'} {volume === 0 ? 'fa-volume-xmark' : ''}"></i>
            </button>
            <div class="slider" style="width: {volume * 100}%">
                <input type="range" min={0} max={1} step={0.01} bind:value={volume} on:input={(e)=>{ audio.volume = e.target.value}}>
            </div>
        </div>
        <label for="upload" class="action-btn fa-solid fa-plus"></label>
        <input multiple type="file" class="hidden" id="upload" on:change={handleFileInput} accept="audio/*"/>
    </navigation>
    <div class="playlist">
        <div class="flex justify-center rounded-3xl bg-red-700 h-7">
        <form on:submit={submitSearch} class="w-1/2 text-center rounded-3xl focus:outline-none bg-gray-950 text-white">
            <input type="search" bind:value={form.searchString}
            class="w-9/12 text-center rounded-3xl focus:outline-none bg-gray-950 text-white placeholder-white" 
            placeholder="Search by a track name"
            aria-label="Search"
            aria-describedby="button-addon1" />
            <button
                class=""
                type="submit"
                id="search_button"
                data-te-ripple-init
                data-te-ripple-color="light">
                <svg
                    xmlns="http://www.w3.org/2000/svg"
                    viewBox="0 0 20 20"
                    fill="currentColor"
                    class="h-5 w-5">
                    <path
                    fill-rule="evenodd"
                    d="M9 3.5a5.5 5.5 0 100 11 5.5 5.5 0 000-11zM2 9a7 7 0 1112.452 4.391l3.328 3.329a.75.75 0 11-1.06 1.06l-3.329-3.328A7 7 0 012 9z"
                    clip-rule="evenodd" />
                </svg>
            </button>
        </div>
        {#each tracks as track}
        <div class="playlist-elements">
            <div on:pointerdown={()=>{getTrack(track)}} class="playsongs">
                <span>{track.name}</span>
            </div>
            <span class="flex items-center justify-center text-white font-mono font-bold">{track.size}</span>
            <button on:click={() => likeAudio(track)} class="fa-{isLiked? 'solid' : 'regular'} fa-heart action-btn {isLiked? 'text-4xl' : 'text-3xl'}"></button>
            <button on:click={() => deleteTrack(track)} class="fa-solid fa-xmark action-btn"></button>
        </div>
        {/each}
        {#each likedtracks as likedtrack}
            <span class="text-white text-6xl">{likedtrack.name}</span>
        {/each}
    </div>
</player> 