<script lang="ts">
    let tracks: File[] = [];
    let likedTracks: File[] = [];
    let audios: HTMLAudioElement[] = [];
    let songIndex: number = 0;
    let audio: HTMLAudioElement;
    let isHidden: boolean = false;
    let isLikedHidden: boolean = true;
    let isLoaded: boolean = false;
    let isLooped: boolean = false;
    let isLiked: boolean = false;
    let isMuted: boolean = false;
    let isNavHidden: boolean = true;
    let isPlaying: boolean = false;
    let volume: number = 0.5;
    let duration: number;
    let totalTimeDisplay = "";
	let currTimeDisplay = "";
    let progress: number = 0;
    let trackTimer: number;
    let searchString:string = '';
    let form = {
    searchString: ''
    };
    let highlight:string = '';

    $: isPlaying ? updateProgress() : null;

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
        audios = [...audios, audio];
        console.log(audio);
    };

    function getTrack(track: Blob | MediaSource){
        audio.pause();
        audio = new Audio(URL.createObjectURL(track));
        highlight = 'bg-red-600';
    };

    function playPauseAudio(){
        toggleTimeRunning();
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
        if(songIndex < 0){
            songIndex = tracks.length - 1;
        }
        if(!isPlaying){
            isPlaying = true;
        }
        audio.pause();
        audio = new Audio(URL.createObjectURL(tracks[songIndex]));
        audio.play();
    };

    function nextAudio(){
        songIndex++;
        if(songIndex > tracks.length - 1){
            songIndex = 0;
        }
        audio.pause()
        if(!isPlaying){
            isPlaying = true;
        }
        audio = new Audio(URL.createObjectURL(tracks[songIndex]));
        audio.play();
    };

    function setProgress(){
        if (audio){
            audio.currentTime = progress / 100 * audio.duration;
        }
    };
    
    function updateProgress(){
		duration = audio.duration;
        progress = audio.currentTime * (100 / duration);

		let currMins = Math.floor(audio.currentTime / 60);
		let currSecs = Math.floor(audio.currentTime - currMins * 60);
		
		let durMins = Math.floor( (duration / 60) % 60 );
		let durSecs =  Math.floor(duration - (durMins * 60));
		
		if(currSecs < 10) currSecs = `0${currSecs}`;
		if(durSecs < 10) durSecs = `0${durSecs}`;
		if(currMins < 10) currMins = `${currMins}`;
		if(durMins < 10) durMins = `${durMins}`;
		
		if (audio.ended){
            nextAudio();
		}
        currTimeDisplay = `${currMins}:${currSecs}`;
        totalTimeDisplay = `${durMins}:${durSecs}`;
    };

    const toggleTimeRunning = () => {
		if (audio.ended) {
			isPlaying = false;
			clearInterval(trackTimer);
		} else {
			trackTimer = setInterval(updateProgress, 50);
		}
	};

    function likeAudio(track: File){
        isLiked = !isLiked;
        likedTracks.push(track);
    };

    function deleteTrack(track: File){
        tracks = tracks.filter(t => t !== track);
    };
</script>
<navbar class="flex bg-zinc-800 font-rubik">
    <sidepanel class="{isNavHidden? 'hidden' : 'max-sm:z-20 max-sm:absolute max-sm:bg-black'} flex flex-col justify-center items-center w-1/4 sm:w-1/12 h-screen gap-10 xl:gap-16 2xl:gap-44 rounded-tr-3xl rounded-br-3xl opacity-95 pt-3">
        <button on:click={()=> {isNavHidden = !isNavHidden}} class="button bg-pink-800 p-3 opacity-95">
            <svg class="svg" xmlns="http://www.w3.org/2000/svg" fill-rule="evenodd" clip-rule="evenodd">
                <path d="M14.8 21.314c-1.185.44-2.463.686-3.8.686-6.075 0-11-4.924-11-11 0-6.074 4.925-11 11-11 6.074 0 10.999 4.926 10.999 11 0 .463-.032.918-.09 1.365.264.282.421.666.421 1.103v.024l.121-.005c.881 0 1.547.675 1.547 1.57l.002 5.839c0 1.914-1.213 3.104-3.167 3.104h-2.64c-2.07 0-2.76-1.406-3.094-2.084l-.299-.602zm6.033 1.686c1.418 0 2.167-.766 2.167-2.104l-.002-5.839c0-.386-.274-.57-.547-.57-.276 0-.549.185-.549.535v2.56c0 .175-.141.259-.281.259-.145 0-.291-.089-.291-.259v-4.114c0-.396-.288-.589-.577-.589-.293 0-.587.199-.587.589v3.884c0 .164-.131.245-.264.245s-.268-.084-.268-.25v-4.557c0-.391-.285-.59-.613-.59-.322 0-.572.193-.572.59l-.002 4.609c0 .183-.143.275-.285.275-.145 0-.287-.092-.287-.275v-3.872c0-.369-.284-.557-.565-.557-.277 0-.553.182-.553.557v4.794c0 .154-.111.234-.226.234-.084 0-.172-.042-.219-.128l-.785-1.444c-.152-.292-.432-.419-.709-.419-.41 0-.818.277-.818.711 0 .111.027.233.085.362.112.265 1.557 3.112 1.913 3.839.375.762.847 1.524 2.195 1.524h2.64zm.124-11.109c.027-.293.042-.59.042-.891 0-5.514-4.486-10-9.999-10-5.514 0-10 4.486-10 10s4.486 10 10 10c1.176 0 2.301-.209 3.348-.587l-.046-.092c-.718-1.433-1.066-2.127-1.125-2.265-.117-.255-.177-.518-.177-.781 0-.943.816-1.711 1.818-1.711.344 0 .664.089.939.254v-2.291c0-.888.668-1.557 1.553-1.557l.326.032c.258-.49.764-.802 1.385-.802.59 0 1.086.28 1.367.72.117-.027.238-.041.365-.041l.204.012zm-12.519 5.991c-1.989-.746-3.569-2.329-4.308-4.321l-.935.116c.821 2.409 2.721 4.311 5.126 5.139l.117-.934zm.238-1.903c-1.166-.547-2.107-1.49-2.654-2.656l-.954.121c.65 1.578 1.91 2.838 3.49 3.489l.118-.954zm2.324-8.645c2.025 0 3.667 1.641 3.667 3.666s-1.642 3.667-3.667 3.667c-2.026 0-3.667-1.642-3.667-3.667s1.641-3.666 3.667-3.666zm0 1c-1.47 0-2.667 1.196-2.667 2.666 0 1.471 1.197 2.667 2.667 2.667 1.471 0 2.667-1.196 2.667-2.667 0-1.47-1.196-2.666-2.667-2.666zm-.916 2.666c0 .506.41.917.916.917s.917-.411.917-.917-.411-.916-.917-.916-.916.41-.916.916zm6.849-2.443c-.65-1.577-1.912-2.838-3.49-3.488l-.119.954c1.166.546 2.107 1.488 2.656 2.654l.953-.12zm.924-.115c-.746-1.977-2.319-3.547-4.301-4.287l.119-.934c2.395.822 4.289 2.712 5.117 5.104l-.935.117z"/>
            </svg>
        </button>
        <div class="flex flex-col gap-6">
            <button on:click={()=> {isLikedHidden? isHidden = false : isLikedHidden=true, isHidden=false, isNavHidden=true}} class="button {isHidden ? '' : 'shadow-[0px_0px_30px_5px_rgba(255,255,255,0.5)]'}">
                <svg xmlns="http://www.w3.org/2000/svg" class="svg" viewBox="0 0 24 24">
                    <path d="M21 13v10h-6v-6h-6v6h-6v-10h-3l12-12 12 12h-3zm-1-5.907v-5.093h-3v2.093l3 3z"/>
                </svg>
            </button>
            <button on:click={()=> {isHidden = true, isLikedHidden = true, isNavHidden=true}} class="button {isHidden && isLikedHidden ? 'shadow-[0px_0px_30px_5px_rgba(255,255,255,0.5)]' : ''}">
                <svg xmlns="http://www.w3.org/2000/svg" class="svg" fill-rule="evenodd" clip-rule="evenodd">
                    <path d="M22 18.318c-.003 1.186-.717 2.414-1.948 3.127-1.731 1.003-3.832.609-4.689-.881-.856-1.489-.146-3.512 1.585-4.516 1.391-.806 3.021-.709 4.052.133v-10.967l-12 2.572v12.475c.02 1.203-.697 2.46-1.948 3.184-1.731 1.003-3.832.609-4.689-.881-.856-1.489-.146-3.512 1.585-4.516 1.391-.806 3.021-.709 4.052.133v-15.181l14-3v18.318zm-17.556.582c1.259-.729 2.756-.496 3.34.52.585 1.017.038 2.434-1.221 3.164-1.26.73-2.757.497-3.341-.52-.585-1.016-.038-2.434 1.222-3.164zm13-2c1.259-.729 2.756-.496 3.34.52.585 1.017.038 2.434-1.221 3.164-1.26.73-2.757.497-3.341-.52-.585-1.016-.038-2.434 1.222-3.164zm-8.444-13.092v2.955l12-2.571v-2.955l-12 2.571z"/>
                </svg>
            </button>
            <button class="button">
                <svg xmlns="http://www.w3.org/2000/svg" class="svg" fill-rule="evenodd" clip-rule="evenodd">
                    <path d="M11 5h13v17h-24v-20h8l3 3zm-10-2v18h22v-15h-12.414l-3-3h-6.586z"/>
                </svg>
            </button>
            <button class="button">
                <svg xmlns="http://www.w3.org/2000/svg" class="svg" viewBox="0 0 24 24">
                    <path d="M12 2c2.757 0 5 2.243 5 5.001 0 2.756-2.243 5-5 5s-5-2.244-5-5c0-2.758 2.243-5.001 5-5.001zm0-2c-3.866 0-7 3.134-7 7.001 0 3.865 3.134 7 7 7s7-3.135 7-7c0-3.867-3.134-7.001-7-7.001zm6.369 13.353c-.497.498-1.057.931-1.658 1.302 2.872 1.874 4.378 5.083 4.972 7.346h-19.387c.572-2.29 2.058-5.503 4.973-7.358-.603-.374-1.162-.811-1.658-1.312-4.258 3.072-5.611 8.506-5.611 10.669h24c0-2.142-1.44-7.557-5.631-10.647z"/>
                </svg>
            </button>
            <div class="flex justify-center flex-col w-full my-10 border-t-2 border-[#fcfcff] border-solid"></div>
            <button on:click={() => {isLikedHidden = false, isHidden = true, isNavHidden=true}} class="button {isLikedHidden ? '' : 'shadow-[0px_0px_30px_5px_rgba(255,255,255,0.5)]'}">
                <svg xmlns="http://www.w3.org/2000/svg" class="svg" fill-rule="evenodd" clip-rule="evenodd" stroke-linejoin="round" stroke-miterlimit="2" viewBox="0 0 24 24">
                    <path d="m7.234 3.004c-2.652 0-5.234 1.829-5.234 5.177 0 3.725 4.345 7.727 9.303 12.54.194.189.446.283.697.283s.503-.094.697-.283c4.977-4.831 9.303-8.814 9.303-12.54 0-3.353-2.58-5.168-5.229-5.168-1.836 0-3.646.866-4.771 2.554-1.13-1.696-2.935-2.563-4.766-2.563zm0 1.5c1.99.001 3.202 1.353 4.155 2.7.14.198.368.316.611.317.243 0 .471-.117.612-.314.955-1.339 2.19-2.694 4.159-2.694 1.796 0 3.729 1.148 3.729 3.668 0 2.671-2.881 5.673-8.5 11.127-5.454-5.285-8.5-8.389-8.5-11.127 0-1.125.389-2.069 1.124-2.727.673-.604 1.625-.95 2.61-.95z" fill-rule="nonzero"/>
                </svg>
            </button>
            <button class="button">
                <svg xmlns="http://www.w3.org/2000/svg" class="svg" fill-rule="evenodd" clip-rule="evenodd">
                    <path d="M15.668 8.626l8.332 1.159-6.065 5.874 1.48 8.341-7.416-3.997-7.416 3.997 1.481-8.341-6.064-5.874 8.331-1.159 3.668-7.626 3.669 7.626zm-6.67.925l-6.818.948 4.963 4.807-1.212 6.825 6.068-3.271 6.069 3.271-1.212-6.826 4.964-4.806-6.819-.948-3.002-6.241-3.001 6.241z"/>
                </svg>
            </button>
            <button class="button">
                <svg xmlns="http://www.w3.org/2000/svg" class="svg" fill-rule="evenodd" clip-rule="evenodd">
                    <path d="M11 5h13v17h-24v-20h8l3 3zm-8 16v-12h20v-3h-12.414l-3-3h-6.586v18h2zm1 0h19v-11h-19v11z"/>
                </svg>
            </button>
        </div>
        <button class="button">
            <svg xmlns="http://www.w3.org/2000/svg" class="svg" fill-rule="evenodd" clip-rule="evenodd">
                <path d="M12 8.666c-1.838 0-3.333 1.496-3.333 3.334s1.495 3.333 3.333 3.333 3.333-1.495 3.333-3.333-1.495-3.334-3.333-3.334m0 7.667c-2.39 0-4.333-1.943-4.333-4.333s1.943-4.334 4.333-4.334 4.333 1.944 4.333 4.334c0 2.39-1.943 4.333-4.333 4.333m-1.193 6.667h2.386c.379-1.104.668-2.451 2.107-3.05 1.496-.617 2.666.196 3.635.672l1.686-1.688c-.508-1.047-1.266-2.199-.669-3.641.567-1.369 1.739-1.663 3.048-2.099v-2.388c-1.235-.421-2.471-.708-3.047-2.098-.572-1.38.057-2.395.669-3.643l-1.687-1.686c-1.117.547-2.221 1.257-3.642.668-1.374-.571-1.656-1.734-2.1-3.047h-2.386c-.424 1.231-.704 2.468-2.099 3.046-.365.153-.718.226-1.077.226-.843 0-1.539-.392-2.566-.893l-1.687 1.686c.574 1.175 1.251 2.237.669 3.643-.571 1.375-1.734 1.654-3.047 2.098v2.388c1.226.418 2.468.705 3.047 2.098.581 1.403-.075 2.432-.669 3.643l1.687 1.687c1.45-.725 2.355-1.204 3.642-.669 1.378.572 1.655 1.738 2.1 3.047m3.094 1h-3.803c-.681-1.918-.785-2.713-1.773-3.123-1.005-.419-1.731.132-3.466.952l-2.689-2.689c.873-1.837 1.367-2.465.953-3.465-.412-.991-1.192-1.087-3.123-1.773v-3.804c1.906-.678 2.712-.782 3.123-1.773.411-.991-.071-1.613-.953-3.466l2.689-2.688c1.741.828 2.466 1.365 3.465.953.992-.412 1.082-1.185 1.775-3.124h3.802c.682 1.918.788 2.714 1.774 3.123 1.001.416 1.709-.119 3.467-.952l2.687 2.688c-.878 1.847-1.361 2.477-.952 3.465.411.992 1.192 1.087 3.123 1.774v3.805c-1.906.677-2.713.782-3.124 1.773-.403.975.044 1.561.954 3.464l-2.688 2.689c-1.728-.82-2.467-1.37-3.456-.955-.988.41-1.08 1.146-1.785 3.126"/>
            </svg>
        </button>
    </sidepanel>
    <content>
        <liked class="{isLikedHidden ? 'hidden' : 'block'} absolute bg-black w-full h-full z-[11] rounded-lg {isNavHidden? '' : 'sm:w-11/12 sm:rounded-2xl'}">
            <div class="flex justify-start">
                <div class="{isNavHidden === true && isLikedHidden === false ? 'block' : 'hidden'}">
                    <button on:click={()=>{isNavHidden=!isNavHidden}} class="button bg-pink-800 p-3 mt-2 ml-2 opacity-95 {isNavHidden? 'block':'md:hidden'}">
                        <svg class="svg" xmlns="http://www.w3.org/2000/svg" fill-rule="evenodd" clip-rule="evenodd">
                            <path d="M14.8 21.314c-1.185.44-2.463.686-3.8.686-6.075 0-11-4.924-11-11 0-6.074 4.925-11 11-11 6.074 0 10.999 4.926 10.999 11 0 .463-.032.918-.09 1.365.264.282.421.666.421 1.103v.024l.121-.005c.881 0 1.547.675 1.547 1.57l.002 5.839c0 1.914-1.213 3.104-3.167 3.104h-2.64c-2.07 0-2.76-1.406-3.094-2.084l-.299-.602zm6.033 1.686c1.418 0 2.167-.766 2.167-2.104l-.002-5.839c0-.386-.274-.57-.547-.57-.276 0-.549.185-.549.535v2.56c0 .175-.141.259-.281.259-.145 0-.291-.089-.291-.259v-4.114c0-.396-.288-.589-.577-.589-.293 0-.587.199-.587.589v3.884c0 .164-.131.245-.264.245s-.268-.084-.268-.25v-4.557c0-.391-.285-.59-.613-.59-.322 0-.572.193-.572.59l-.002 4.609c0 .183-.143.275-.285.275-.145 0-.287-.092-.287-.275v-3.872c0-.369-.284-.557-.565-.557-.277 0-.553.182-.553.557v4.794c0 .154-.111.234-.226.234-.084 0-.172-.042-.219-.128l-.785-1.444c-.152-.292-.432-.419-.709-.419-.41 0-.818.277-.818.711 0 .111.027.233.085.362.112.265 1.557 3.112 1.913 3.839.375.762.847 1.524 2.195 1.524h2.64zm.124-11.109c.027-.293.042-.59.042-.891 0-5.514-4.486-10-9.999-10-5.514 0-10 4.486-10 10s4.486 10 10 10c1.176 0 2.301-.209 3.348-.587l-.046-.092c-.718-1.433-1.066-2.127-1.125-2.265-.117-.255-.177-.518-.177-.781 0-.943.816-1.711 1.818-1.711.344 0 .664.089.939.254v-2.291c0-.888.668-1.557 1.553-1.557l.326.032c.258-.49.764-.802 1.385-.802.59 0 1.086.28 1.367.72.117-.027.238-.041.365-.041l.204.012zm-12.519 5.991c-1.989-.746-3.569-2.329-4.308-4.321l-.935.116c.821 2.409 2.721 4.311 5.126 5.139l.117-.934zm.238-1.903c-1.166-.547-2.107-1.49-2.654-2.656l-.954.121c.65 1.578 1.91 2.838 3.49 3.489l.118-.954zm2.324-8.645c2.025 0 3.667 1.641 3.667 3.666s-1.642 3.667-3.667 3.667c-2.026 0-3.667-1.642-3.667-3.667s1.641-3.666 3.667-3.666zm0 1c-1.47 0-2.667 1.196-2.667 2.666 0 1.471 1.197 2.667 2.667 2.667 1.471 0 2.667-1.196 2.667-2.667 0-1.47-1.196-2.666-2.667-2.666zm-.916 2.666c0 .506.41.917.916.917s.917-.411.917-.917-.411-.916-.917-.916-.916.41-.916.916zm6.849-2.443c-.65-1.577-1.912-2.838-3.49-3.488l-.119.954c1.166.546 2.107 1.488 2.656 2.654l.953-.12zm.924-.115c-.746-1.977-2.319-3.547-4.301-4.287l.119-.934c2.395.822 4.289 2.712 5.117 5.104l-.935.117z"/>
                        </svg>
                    </button>
                </div>
                <h4 class="flex justify-start font-semibold text-[#fcfcff] text-4xl px-2 pt-2">Liked songs</h4>
            </div>
            <div class="flex flex-col h-full w-full pl-2 gap-2 md:pl-4 overflow-hidden ">
                {#each likedTracks as ltrack}
                    <div class="flex justify-start items-center w-full h-16 max-[375px]:gap-2 gap-10 md:gap-14 mt-1 bg-black text-[#fcfcff] rounded-xl">
                        <div class="flex justify-start items-center w-2/3 gap-2 md:gap-3">
                            <span class="text-bold text-xl px-2"></span>
                            <div class="flex bg-red-700 h-14 w-14 rounded-xl md:rounded-2xl"></div>
                            <div class="flex w-4/5 flex-col pl-1">
                                <span class="font-bold overflow-hidden">{ltrack.name}</span>
                                <span>🧑‍🎤artist</span>
                            </div>
                        </div>
                        <span>3:36</span>
                        <div class="flex justify-center max-[375px]:gap-2 gap-6 pr-4">
                            <button  class="button">
                                <svg clip-rule="evenodd" class="svg fill-[#fcfcff]" fill-rule="evenodd" stroke-linejoin="round" stroke-miterlimit="2" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                                    <path d="{isLiked === true && tracks.length > 0 ? 'm12 5.72c-2.624-4.517-10-3.198-10 2.461 0 3.725 4.345 7.727 9.303 12.54.194.189.446.283.697.283s.503-.094.697-.283c4.977-4.831 9.303-8.814 9.303-12.54 0-5.678-7.396-6.944-10-2.461z' 
                                    :'m7.234 3.004c-2.652 0-5.234 1.829-5.234 5.177 0 3.725 4.345 7.727 9.303 12.54.194.189.446.283.697.283s.503-.094.697-.283c4.977-4.831 9.303-8.814 9.303-12.54 0-3.353-2.58-5.168-5.229-5.168-1.836 0-3.646.866-4.771 2.554-1.13-1.696-2.935-2.563-4.766-2.563zm0 1.5c1.99.001 3.202 1.353 4.155 2.7.14.198.368.316.611.317.243 0 .471-.117.612-.314.955-1.339 2.19-2.694 4.159-2.694 1.796 0 3.729 1.148 3.729 3.668 0 2.671-2.881 5.673-8.5 11.127-5.454-5.285-8.5-8.389-8.5-11.127 0-1.125.389-2.069 1.124-2.727.673-.604 1.625-.95 2.61-.95z'} " fill-rule="nonzero"/>
                                </svg>
                            </button>
                            <button  class="button">
                                <svg clip-rule="evenodd" class="svg fill-[#fcfcff]" fill-rule="evenodd" stroke-linejoin="round" stroke-miterlimit="2" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                                    <path d="m16.5 11.995c0-1.242 1.008-2.25 2.25-2.25s2.25 1.008 2.25 2.25-1.008 2.25-2.25 2.25-2.25-1.008-2.25-2.25zm-6.75 0c0-1.242 1.008-2.25 2.25-2.25s2.25 1.008 2.25 2.25-1.008 2.25-2.25 2.25-2.25-1.008-2.25-2.25zm-6.75 0c0-1.242 1.008-2.25 2.25-2.25s2.25 1.008 2.25 2.25-1.008 2.25-2.25 2.25-2.25-1.008-2.25-2.25z"/>
                                </svg>
                            </button>
                        </div>
                    </div>
                {/each}
            </div>
        </liked>
        <homepage class="flex justify-center z-10 absolute right-0 w-full rounded-lg {isNavHidden? '' : 'sm:w-11/12 sm:rounded-2xl'} h-full bg-[#0d0c22]"> 
            <img src="img/stars_background.png" alt="" class="hidden md:flex justify-center absolute z-0 h-full w-full">
            <div class="{isNavHidden === true && isHidden === true ? 'block' : 'hidden'}">
                <button on:click={()=>{isNavHidden=!isNavHidden}} class="button absolute left-0 bg-pink-800 p-3 mt-2 ml-2 opacity-95 {isNavHidden? 'block':'md:hidden'}">
                    <svg class="svg" xmlns="http://www.w3.org/2000/svg" fill-rule="evenodd" clip-rule="evenodd">
                        <path d="M14.8 21.314c-1.185.44-2.463.686-3.8.686-6.075 0-11-4.924-11-11 0-6.074 4.925-11 11-11 6.074 0 10.999 4.926 10.999 11 0 .463-.032.918-.09 1.365.264.282.421.666.421 1.103v.024l.121-.005c.881 0 1.547.675 1.547 1.57l.002 5.839c0 1.914-1.213 3.104-3.167 3.104h-2.64c-2.07 0-2.76-1.406-3.094-2.084l-.299-.602zm6.033 1.686c1.418 0 2.167-.766 2.167-2.104l-.002-5.839c0-.386-.274-.57-.547-.57-.276 0-.549.185-.549.535v2.56c0 .175-.141.259-.281.259-.145 0-.291-.089-.291-.259v-4.114c0-.396-.288-.589-.577-.589-.293 0-.587.199-.587.589v3.884c0 .164-.131.245-.264.245s-.268-.084-.268-.25v-4.557c0-.391-.285-.59-.613-.59-.322 0-.572.193-.572.59l-.002 4.609c0 .183-.143.275-.285.275-.145 0-.287-.092-.287-.275v-3.872c0-.369-.284-.557-.565-.557-.277 0-.553.182-.553.557v4.794c0 .154-.111.234-.226.234-.084 0-.172-.042-.219-.128l-.785-1.444c-.152-.292-.432-.419-.709-.419-.41 0-.818.277-.818.711 0 .111.027.233.085.362.112.265 1.557 3.112 1.913 3.839.375.762.847 1.524 2.195 1.524h2.64zm.124-11.109c.027-.293.042-.59.042-.891 0-5.514-4.486-10-9.999-10-5.514 0-10 4.486-10 10s4.486 10 10 10c1.176 0 2.301-.209 3.348-.587l-.046-.092c-.718-1.433-1.066-2.127-1.125-2.265-.117-.255-.177-.518-.177-.781 0-.943.816-1.711 1.818-1.711.344 0 .664.089.939.254v-2.291c0-.888.668-1.557 1.553-1.557l.326.032c.258-.49.764-.802 1.385-.802.59 0 1.086.28 1.367.72.117-.027.238-.041.365-.041l.204.012zm-12.519 5.991c-1.989-.746-3.569-2.329-4.308-4.321l-.935.116c.821 2.409 2.721 4.311 5.126 5.139l.117-.934zm.238-1.903c-1.166-.547-2.107-1.49-2.654-2.656l-.954.121c.65 1.578 1.91 2.838 3.49 3.489l.118-.954zm2.324-8.645c2.025 0 3.667 1.641 3.667 3.666s-1.642 3.667-3.667 3.667c-2.026 0-3.667-1.642-3.667-3.667s1.641-3.666 3.667-3.666zm0 1c-1.47 0-2.667 1.196-2.667 2.666 0 1.471 1.197 2.667 2.667 2.667 1.471 0 2.667-1.196 2.667-2.667 0-1.47-1.196-2.666-2.667-2.666zm-.916 2.666c0 .506.41.917.916.917s.917-.411.917-.917-.411-.916-.917-.916-.916.41-.916.916zm6.849-2.443c-.65-1.577-1.912-2.838-3.49-3.488l-.119.954c1.166.546 2.107 1.488 2.656 2.654l.953-.12zm.924-.115c-.746-1.977-2.319-3.547-4.301-4.287l.119-.934c2.395.822 4.289 2.712 5.117 5.104l-.935.117z"/>
                    </svg>
                </button>
                <div class="flex flex-col items-center h-4/5 w-full gap-10">
                    <div class="flex justify-center items-center h-full w-full p-5">
                        <img src="img/stars_background.png" alt="" class=" w-fit h-full rounded-3xl">
                        <div class="bg-red-700 h-1/3 w-3/4 absolute sm:h-1/2 md:hidden"></div>
                    </div>
                    <div class="flex justify-center items-center flex-col p-4 text-white w-full">
                        <span class="font-bold text-xl truncate w-[200px] min-[310px]:w-[300px]">{tracks.length > 0 ? tracks[songIndex].name : ''}</span>
                        <span>🧑‍🎤artist</span>
                    </div>
                    <div class="{tracks.length > 0 ? 'flex' : 'hidden'} justify-center items-center gap-1 w-5/6 md:hidden">
                        <span class="text-white mr-3 w-5">{currTimeDisplay}</span>
                        <div class="relative w-[80%] rounded-md cursor-pointer h-2">
                            <div class="progress" style="width: {progress}%"></div>
                            <input type="range" class="seek-slider" min={0} max={100} step={1} bind:value={progress} on:input={setProgress}>
                        </div>
                        <span class="text-white w-[10%]">{totalTimeDisplay}</span>
                    </div>
                </div>
            </div>
            <playlist class="{isHidden ? 'hidden': 'block'} flex w-full h-full bg-black rounded-2xl z-0">
                <div class="flex flex-col w-full md:w-[70%]">
                    <div class="flex justify-start items-center">
                        <button on:click={()=>{isNavHidden=!isNavHidden}} class="button bg-pink-800 p-3 mt-2 ml-2 opacity-95 {isNavHidden? 'block':'md:hidden'}">
                            <svg class="svg" xmlns="http://www.w3.org/2000/svg" fill-rule="evenodd" clip-rule="evenodd">
                                <path d="M14.8 21.314c-1.185.44-2.463.686-3.8.686-6.075 0-11-4.924-11-11 0-6.074 4.925-11 11-11 6.074 0 10.999 4.926 10.999 11 0 .463-.032.918-.09 1.365.264.282.421.666.421 1.103v.024l.121-.005c.881 0 1.547.675 1.547 1.57l.002 5.839c0 1.914-1.213 3.104-3.167 3.104h-2.64c-2.07 0-2.76-1.406-3.094-2.084l-.299-.602zm6.033 1.686c1.418 0 2.167-.766 2.167-2.104l-.002-5.839c0-.386-.274-.57-.547-.57-.276 0-.549.185-.549.535v2.56c0 .175-.141.259-.281.259-.145 0-.291-.089-.291-.259v-4.114c0-.396-.288-.589-.577-.589-.293 0-.587.199-.587.589v3.884c0 .164-.131.245-.264.245s-.268-.084-.268-.25v-4.557c0-.391-.285-.59-.613-.59-.322 0-.572.193-.572.59l-.002 4.609c0 .183-.143.275-.285.275-.145 0-.287-.092-.287-.275v-3.872c0-.369-.284-.557-.565-.557-.277 0-.553.182-.553.557v4.794c0 .154-.111.234-.226.234-.084 0-.172-.042-.219-.128l-.785-1.444c-.152-.292-.432-.419-.709-.419-.41 0-.818.277-.818.711 0 .111.027.233.085.362.112.265 1.557 3.112 1.913 3.839.375.762.847 1.524 2.195 1.524h2.64zm.124-11.109c.027-.293.042-.59.042-.891 0-5.514-4.486-10-9.999-10-5.514 0-10 4.486-10 10s4.486 10 10 10c1.176 0 2.301-.209 3.348-.587l-.046-.092c-.718-1.433-1.066-2.127-1.125-2.265-.117-.255-.177-.518-.177-.781 0-.943.816-1.711 1.818-1.711.344 0 .664.089.939.254v-2.291c0-.888.668-1.557 1.553-1.557l.326.032c.258-.49.764-.802 1.385-.802.59 0 1.086.28 1.367.72.117-.027.238-.041.365-.041l.204.012zm-12.519 5.991c-1.989-.746-3.569-2.329-4.308-4.321l-.935.116c.821 2.409 2.721 4.311 5.126 5.139l.117-.934zm.238-1.903c-1.166-.547-2.107-1.49-2.654-2.656l-.954.121c.65 1.578 1.91 2.838 3.49 3.489l.118-.954zm2.324-8.645c2.025 0 3.667 1.641 3.667 3.666s-1.642 3.667-3.667 3.667c-2.026 0-3.667-1.642-3.667-3.667s1.641-3.666 3.667-3.666zm0 1c-1.47 0-2.667 1.196-2.667 2.666 0 1.471 1.197 2.667 2.667 2.667 1.471 0 2.667-1.196 2.667-2.667 0-1.47-1.196-2.666-2.667-2.666zm-.916 2.666c0 .506.41.917.916.917s.917-.411.917-.917-.411-.916-.917-.916-.916.41-.916.916zm6.849-2.443c-.65-1.577-1.912-2.838-3.49-3.488l-.119.954c1.166.546 2.107 1.488 2.656 2.654l.953-.12zm.924-.115c-.746-1.977-2.319-3.547-4.301-4.287l.119-.934c2.395.822 4.289 2.712 5.117 5.104l-.935.117z"/>
                            </svg>
                        </button>
                        <h4 class="font-semibold text-[#fcfcff] text-4xl px-2 pt-2">Home</h4>
                        <div class="flex flex-col w-full md:hidden">
                            <div class="relative">
                                <button class="button absolute pt-9">
                                    <svg xmlns="http://www.w3.org/2000/svg" class="svg fill-[#fcfcff]" viewBox="0 0 24 24">
                                        <path d="M12 14.5c0-3.033 2.081-5.574 4.891-6.291-.476-1.777-1.209-3.367-2.753-4.264-.644-.374-1.042-1.07-1.041-1.82v-.003c0-1.172-.939-2.122-2.097-2.122s-2.097.95-2.097 2.122v.003c.001.751-.396 1.446-1.041 1.82-4.667 2.712-1.985 11.715-6.862 13.306v1.749h12.816c-1.122-1.168-1.816-2.752-1.816-4.5zm-1-11.5c-.552 0-1-.448-1-1 0-.551.448-1 1-1s1 .449 1 1c0 .552-.448 1-1 1zm-3 18h6c0 1.598-1.392 3-2.971 3s-3.029-1.402-3.029-3zm10.5-11c-2.486 0-4.5 2.015-4.5 4.5s2.014 4.5 4.5 4.5c2.484 0 4.5-2.015 4.5-4.5s-2.016-4.5-4.5-4.5zm-.469 6.484l-1.688-1.637.695-.697.992.94 2.115-2.169.697.696-2.811 2.867z"/>
                                    </svg>
                                </button>
                                <div class="pt-8 ml-12">
                                    <form on:submit={submitSearch} class="relative mb-4 flex w-11/12 flex-wrap items-stretch">
                                    <input type="search" bind:value={form.searchString} class="relative m-0 block w-[1px] min-w-0 flex-auto rounded-md placeholder-gray-400 border border-solid border-neutral-300 bg-transparent bg-clip-padding px-7 py-[0.25rem] text-sm font-normal leading-[1.6] text-neutral-700 outline-none transition duration-200 ease-in-out focus:z-[3] focus:border-primary focus:text-neutral-700 focus:shadow-[inset_0_0_0_1px_rgb(59,113,202)] focus:outline-none dark:border-neutral-600 dark:text-neutral-200 dark:placeholder:text-neutral-200 dark:focus:border-primary" 
                                        placeholder="Search" aria-label="Search" aria-describedby="button-addon2" />
                                    <span class="input-group-text flex items-center whitespace-nowrap rounded px-3 py-1.5 text-center text-base font-normal text-neutral-700 dark:text-neutral-200">
                                        <svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4 ml-2 absolute left-0 fill-[#fcfcff]" viewBox="0 0 20 20" fill="currentColor">
                                            <path fill-rule="evenodd" d="M9 3.5a5.5 5.5 0 100 11 5.5 5.5 0 000-11zM2 9a7 7 0 1112.452 4.391l3.328 3.329a.75.75 0 11-1.06 1.06l-3.329-3.328A7 7 0 012 9z" clip-rule="evenodd" />
                                        </svg>
                                    </span>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="flex justify-center h-[70%]">
                        <div class="bg-red-700 w-[80%] md:w-[87%] h-full my-2 rounded-xl"></div>
                    </div>
                    <div class="flex justify-between p-4">
                        <h4 class="font-medium text-[#fcfcff] text-xl">Uploaded songs</h4>
                        <a href="" class="underline text-blue-900">See all</a>
                    </div>
                    <div class="flex flex-col h-full w-full pl-2 gap-2 md:pl-4 overflow-y-scroll ">
                        {#each tracks as track, i}
                            <div on:pointerdown={()=>{getTrack(track)}} class="flex justify-start items-center w-full h-16 max-[375px]:gap-0 gap-10 md:gap-14 mt-1 {highlight} bg-black text-[#fcfcff] rounded-xl">
                                <div class="flex justify-start items-center w-2/3 gap-2 md:gap-3">
                                    <span class="text-bold text-xl px-2 hidden sm:block">{i<9 ?'0':''}{i+1}</span>
                                    <div class="flex bg-red-700 h-14 w-14 rounded-xl md:rounded-2xl"></div>
                                    <div class="flex w-4/5 flex-col pl-1">
                                        <span class="font-bold truncate">{track.name}</span>
                                        <span>🧑‍🎤artist</span>
                                    </div>
                                </div>
                                    <span class="flex w-1/12">{audios[i].duration}</span>
                                <div class="flex justify-center max-[375px]:gap-2 gap-6 pr-4">
                                    <button on:click={() => likeAudio(track)} class="button">
                                        <svg clip-rule="evenodd" class="svg fill-[#fcfcff]" fill-rule="evenodd" stroke-linejoin="round" stroke-miterlimit="2" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                                            <path d="{isLiked === true && tracks.length > 0 ? 'm12 5.72c-2.624-4.517-10-3.198-10 2.461 0 3.725 4.345 7.727 9.303 12.54.194.189.446.283.697.283s.503-.094.697-.283c4.977-4.831 9.303-8.814 9.303-12.54 0-5.678-7.396-6.944-10-2.461z' 
                                            :'m7.234 3.004c-2.652 0-5.234 1.829-5.234 5.177 0 3.725 4.345 7.727 9.303 12.54.194.189.446.283.697.283s.503-.094.697-.283c4.977-4.831 9.303-8.814 9.303-12.54 0-3.353-2.58-5.168-5.229-5.168-1.836 0-3.646.866-4.771 2.554-1.13-1.696-2.935-2.563-4.766-2.563zm0 1.5c1.99.001 3.202 1.353 4.155 2.7.14.198.368.316.611.317.243 0 .471-.117.612-.314.955-1.339 2.19-2.694 4.159-2.694 1.796 0 3.729 1.148 3.729 3.668 0 2.671-2.881 5.673-8.5 11.127-5.454-5.285-8.5-8.389-8.5-11.127 0-1.125.389-2.069 1.124-2.727.673-.604 1.625-.95 2.61-.95z'} " fill-rule="nonzero"/>
                                        </svg>
                                    </button>
                                    <button on:click={() => deleteTrack(track)} class="button">
                                        <svg clip-rule="evenodd" class="svg fill-[#fcfcff]" fill-rule="evenodd" stroke-linejoin="round" stroke-miterlimit="2" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                                            <path d="m16.5 11.995c0-1.242 1.008-2.25 2.25-2.25s2.25 1.008 2.25 2.25-1.008 2.25-2.25 2.25-2.25-1.008-2.25-2.25zm-6.75 0c0-1.242 1.008-2.25 2.25-2.25s2.25 1.008 2.25 2.25-1.008 2.25-2.25 2.25-2.25-1.008-2.25-2.25zm-6.75 0c0-1.242 1.008-2.25 2.25-2.25s2.25 1.008 2.25 2.25-1.008 2.25-2.25 2.25-2.25-1.008-2.25-2.25z"/>
                                        </svg>
                                    </button>
                                </div>
                            </div>
                        {/each}
                    </div>
                </div>
                <div class="flex flex-col w-0 max-md:hidden md:w-[30%]">
                    <div class="relative">
                        <button class="button absolute pt-[22px]">
                            <svg xmlns="http://www.w3.org/2000/svg" class="svg fill-[#fcfcff]" viewBox="0 0 24 24">
                                <path d="M12 14.5c0-3.033 2.081-5.574 4.891-6.291-.476-1.777-1.209-3.367-2.753-4.264-.644-.374-1.042-1.07-1.041-1.82v-.003c0-1.172-.939-2.122-2.097-2.122s-2.097.95-2.097 2.122v.003c.001.751-.396 1.446-1.041 1.82-4.667 2.712-1.985 11.715-6.862 13.306v1.749h12.816c-1.122-1.168-1.816-2.752-1.816-4.5zm-1-11.5c-.552 0-1-.448-1-1 0-.551.448-1 1-1s1 .449 1 1c0 .552-.448 1-1 1zm-3 18h6c0 1.598-1.392 3-2.971 3s-3.029-1.402-3.029-3zm10.5-11c-2.486 0-4.5 2.015-4.5 4.5s2.014 4.5 4.5 4.5c2.484 0 4.5-2.015 4.5-4.5s-2.016-4.5-4.5-4.5zm-.469 6.484l-1.688-1.637.695-.697.992.94 2.115-2.169.697.696-2.811 2.867z"/>
                            </svg>
                        </button>
                        <div class="pt-4 ml-12">
                            <form on:submit={submitSearch} class="relative mb-4 flex w-11/12 flex-wrap items-stretch">
                            <input type="search" bind:value={form.searchString} class="relative m-0 block w-[1px] min-w-0 flex-auto rounded-md placeholder-gray-400 border border-solid border-neutral-300 bg-transparent bg-clip-padding px-7 py-[0.25rem] text-sm font-normal leading-[1.6] text-neutral-700 outline-none transition duration-200 ease-in-out focus:z-[3] focus:border-primary focus:text-neutral-700 focus:shadow-[inset_0_0_0_1px_rgb(59,113,202)] focus:outline-none dark:border-neutral-600 dark:text-neutral-200 dark:placeholder:text-neutral-200 dark:focus:border-primary" 
                                placeholder="Search" aria-label="Search" aria-describedby="button-addon2" />
                            <span class="input-group-text flex items-center whitespace-nowrap rounded px-3 py-1.5 text-center text-base font-normal text-neutral-700 dark:text-neutral-200">
                                <svg xmlns="http://www.w3.org/2000/svg" class="w-4 h-4 ml-2 absolute left-0 fill-[#fcfcff]" viewBox="0 0 20 20" fill="currentColor">
                                    <path fill-rule="evenodd" d="M9 3.5a5.5 5.5 0 100 11 5.5 5.5 0 000-11zM2 9a7 7 0 1112.452 4.391l3.328 3.329a.75.75 0 11-1.06 1.06l-3.329-3.328A7 7 0 012 9z" clip-rule="evenodd" />
                                </svg>
                            </span>
                        </div>
                    </div>
                    <div class="flex justify-between p-4">
                        <h4 class="font-medium text-[#fcfcff] text-xl">Top Artist</h4>
                        <a href="" class="underline text-blue-900">See all</a>
                    </div>
                    <div class="flex flex-col h-[204px] overflow-y-scroll">
                        <div class="flex justify-start items-center w-full h-16 gap-14 mt-1 bg-black text-[#fcfcff] rounded-xl">
                            <div class="flex justify-start items-center w-2/3 pl-2">
                                <div class="flex bg-red-700 h-14 w-14 rounded-xl"></div>
                                <div class="flex flex-col pl-1">
                                    <span class="font-bold">🧑‍🎤artist</span>
                                    <span class="text-xs opacity-75">1M followers ♬ 100M plays</span>
                                </div>
                            </div>
                            <div class="flex justify-center gap-6 pr-4">
                                <button class="button">
                                    <svg clip-rule="evenodd" class="svg fill-[#fcfcff]" fill-rule="evenodd" stroke-linejoin="round" stroke-miterlimit="2" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                                        <path d="m16.5 11.995c0-1.242 1.008-2.25 2.25-2.25s2.25 1.008 2.25 2.25-1.008 2.25-2.25 2.25-2.25-1.008-2.25-2.25zm-6.75 0c0-1.242 1.008-2.25 2.25-2.25s2.25 1.008 2.25 2.25-1.008 2.25-2.25 2.25-2.25-1.008-2.25-2.25zm-6.75 0c0-1.242 1.008-2.25 2.25-2.25s2.25 1.008 2.25 2.25-1.008 2.25-2.25 2.25-2.25-1.008-2.25-2.25z"/>
                                    </svg>
                                </button>
                            </div>
                        </div>
                        <div class="flex justify-start items-center w-full h-16 gap-14 mt-1 bg-black text-[#fcfcff] rounded-xl">
                            <div class="flex justify-start items-center w-2/3 pl-2">
                                <div class="flex bg-red-700 h-14 w-14 rounded-xl"></div>
                                <div class="flex flex-col pl-1">
                                    <span class="font-bold">🧑‍🎤artist</span>
                                    <span class="text-xs opacity-75">1M followers ♬ 100M plays</span>
                                </div>
                            </div>
                            <div class="flex justify-center gap-6 pr-4">
                                <button class="button">
                                    <svg clip-rule="evenodd" class="svg fill-[#fcfcff]" fill-rule="evenodd" stroke-linejoin="round" stroke-miterlimit="2" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                                        <path d="m16.5 11.995c0-1.242 1.008-2.25 2.25-2.25s2.25 1.008 2.25 2.25-1.008 2.25-2.25 2.25-2.25-1.008-2.25-2.25zm-6.75 0c0-1.242 1.008-2.25 2.25-2.25s2.25 1.008 2.25 2.25-1.008 2.25-2.25 2.25-2.25-1.008-2.25-2.25zm-6.75 0c0-1.242 1.008-2.25 2.25-2.25s2.25 1.008 2.25 2.25-1.008 2.25-2.25 2.25-2.25-1.008-2.25-2.25z"/>
                                    </svg>
                                </button>
                            </div>
                        </div>
                        <div class="flex justify-start items-center w-full h-16 gap-14 mt-1 bg-black text-[#fcfcff] rounded-xl">
                            <div class="flex justify-start items-center w-2/3 pl-2">
                                <div class="flex bg-red-700 h-14 w-14 rounded-xl"></div>
                                <div class="flex flex-col pl-1">
                                    <span class="font-bold">🧑‍🎤artist</span>
                                    <span class="text-xs opacity-75">1M followers ♬ 100M plays</span>
                                </div>
                            </div>
                            <div class="flex justify-center gap-6 pr-4">
                                <button class="button">
                                    <svg clip-rule="evenodd" class="svg fill-[#fcfcff]" fill-rule="evenodd" stroke-linejoin="round" stroke-miterlimit="2" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                                        <path d="m16.5 11.995c0-1.242 1.008-2.25 2.25-2.25s2.25 1.008 2.25 2.25-1.008 2.25-2.25 2.25-2.25-1.008-2.25-2.25zm-6.75 0c0-1.242 1.008-2.25 2.25-2.25s2.25 1.008 2.25 2.25-1.008 2.25-2.25 2.25-2.25-1.008-2.25-2.25zm-6.75 0c0-1.242 1.008-2.25 2.25-2.25s2.25 1.008 2.25 2.25-1.008 2.25-2.25 2.25-2.25-1.008-2.25-2.25z"/>
                                    </svg>
                                </button>
                            </div>
                        </div>
                    </div>
                    <h4 class="font-medium text-[#fcfcff] text-xl p-2">Recent favourites</h4>
                    <div class="flex gap-2 justify-center px-2 overflow-hidden">
                        <div class="flex flex-col gap-2">
                            <div class="bg-red-600 h-36 w-36 rounded-xl"></div>
                            <div class="bg-red-600 h-36 w-36 rounded-xl"></div>
                        </div>
                        <div class="flex flex-col max-lg:hidden gap-2">
                            <div class="bg-red-600 h-36 w-36 rounded-xl"></div>
                            <div class="bg-red-600 h-36 w-36 rounded-xl"></div>
                        </div>
                    </div>
                </div>
            </playlist>
            <smediabar class="flex items-center justify-between md:hidden lg:justify-start absolute bottom-3 w-[95%] bg-[#0d0c22] p-4 gap-4 md:gap-8 2xl:gap-28 rounded-full opacity-90">
                <button on:click={loopAudio} class="button">
                    <svg xmlns="http://www.w3.org/2000/svg" class="svg {isLooped? 'fill-red-700' : ''}" viewBox="0 0 24 24">
                        <path d="M2 12c0 .999.381 1.902.989 2.604l-1.098.732-.587.392c-.814-1.025-1.304-2.318-1.304-3.728 0-3.313 2.687-6 6-6h9v-3l6 4-6 4v-3h-9c-2.206 0-4 1.794-4 4zm20.696-3.728l-.587.392-1.098.732c.608.702.989 1.605.989 2.604 0 2.206-1.795 4-4 4h-9v-3l-6 4 6 4v-3h9c3.313 0 6-2.687 6-6 0-1.41-.489-2.703-1.304-3.728z"/>
                    </svg>
                </button>
                <div class="flex justify-center gap-2">
                    <button on:pointerdown={prevAudio} class="button">
                        <svg xmlns="http://www.w3.org/2000/svg" class="svg" viewBox="0 0 24 24">
                            <path d="M4 2v20h-2v-20h2zm18 0l-16 10 16 10v-20z"/>
                        </svg>
                    </button>
                    <button on:pointerdown={tracks.length > 0 ? playPauseAudio : null} class="big-button">
                        <img src="{isPlaying ? 'img/pause.png' : 'img/play.png'}" alt="|>" class="svg shadow-[0px_0px_30px_5px_rgba(255,255,255,0.5)]">
                    </button>
                    <button on:pointerdown={nextAudio} class="button">
                        <svg xmlns="http://www.w3.org/2000/svg" class="svg" viewBox="0 0 24 24">
                            <path d="M20 22v-20h2v20h-2zm-18 0l16-10-16-10v20z"/>
                        </svg>
                    </button>
                </div>
                <label for="upload" class="button">
                    <svg xmlns="http://www.w3.org/2000/svg" class="svg" viewBox="0 0 24 24">
                        <path d="M2 7h-2v-2h2c3.49 0 5.48 1.221 6.822 2.854-.41.654-.754 1.312-1.055 1.939-1.087-1.643-2.633-2.793-5.767-2.793zm16 10c-3.084 0-4.604-1.147-5.679-2.786-.302.627-.647 1.284-1.06 1.937 1.327 1.629 3.291 2.849 6.739 2.849v3l6-4-6-4v3zm0-10v3l6-4-6-4v3c-5.834 0-7.436 3.482-8.85 6.556-1.343 2.921-2.504 5.444-7.15 5.444h-2v2h2c5.928 0 7.543-3.511 8.968-6.609 1.331-2.893 2.479-5.391 7.032-5.391z"/>
                    </svg>
                    <input multiple type="file" class="hidden" id="upload" on:change={handleFileInput} accept="audio/*"/>
                </label>
            </smediabar>
            <mediabar class="flex items-center justify-between max-md:hidden absolute bottom-3 w-[95%] bg-[#0d0c22] p-4 gap-11 2xl:gap-28 rounded-full opacity-90">
                <div class="flex justify-center w-1/5 pl-2 gap-2">
                    <button on:pointerdown={prevAudio} class="button">
                        <svg xmlns="http://www.w3.org/2000/svg" class="svg" viewBox="0 0 24 24">
                            <path d="M4 2v20h-2v-20h2zm18 0l-16 10 16 10v-20z"/>
                        </svg>
                    </button>
                    <button on:pointerdown={tracks.length > 0 ? playPauseAudio : null} class="big-button">
                        <img src="{isPlaying ? 'img/pause.png' : 'img/play.png'}" alt="|>" class="svg shadow-[0px_0px_30px_5px_rgba(255,255,255,0.5)]">
                    </button>
                    <button on:pointerdown={nextAudio} class="button">
                        <svg xmlns="http://www.w3.org/2000/svg" class="svg" viewBox="0 0 24 24">
                            <path d="M20 22v-20h2v20h-2zm-18 0l16-10-16-10v20z"/>
                        </svg>
                    </button>
                </div>
                <div class="flex justify-center w-1/5">
                    <div class="flex bg-red-700 h-14 w-14 rounded-xl"></div>
                    <div class="flex flex-col pl-1 text-white overflow-hidden w-5/6">
                        <span class="font-bold truncate">{tracks.length > 0 ? tracks[songIndex].name : ''}</span>
                        <span>🧑‍🎤artist</span>
                    </div>
                </div>
                <div class="flex justify-center gap-4">
                    <button on:click={() => likeAudio(audio)} class="button">
                        <svg clip-rule="evenodd" class="svg" fill-rule="evenodd" stroke-linejoin="round" stroke-miterlimit="2" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                            <path d="{isLiked === true && tracks.length > 0 ? 'm12 5.72c-2.624-4.517-10-3.198-10 2.461 0 3.725 4.345 7.727 9.303 12.54.194.189.446.283.697.283s.503-.094.697-.283c4.977-4.831 9.303-8.814 9.303-12.54 0-5.678-7.396-6.944-10-2.461z' 
                            :'m7.234 3.004c-2.652 0-5.234 1.829-5.234 5.177 0 3.725 4.345 7.727 9.303 12.54.194.189.446.283.697.283s.503-.094.697-.283c4.977-4.831 9.303-8.814 9.303-12.54 0-3.353-2.58-5.168-5.229-5.168-1.836 0-3.646.866-4.771 2.554-1.13-1.696-2.935-2.563-4.766-2.563zm0 1.5c1.99.001 3.202 1.353 4.155 2.7.14.198.368.316.611.317.243 0 .471-.117.612-.314.955-1.339 2.19-2.694 4.159-2.694 1.796 0 3.729 1.148 3.729 3.668 0 2.671-2.881 5.673-8.5 11.127-5.454-5.285-8.5-8.389-8.5-11.127 0-1.125.389-2.069 1.124-2.727.673-.604 1.625-.95 2.61-.95z'} " fill-rule="nonzero"/>
                        </svg>
                    </button>
                    <button class="button">
                        <svg xmlns="http://www.w3.org/2000/svg" class="svg" viewBox="0 0 24 24">
                            <path d="M8 10h-5l9-10 9 10h-5v10h-8v-10zm11 9v3h-14v-3h-2v5h18v-5h-2z"/>
                        </svg>
                    </button>
                    <label for="upload" class="button">
                        <svg xmlns="http://www.w3.org/2000/svg" class="svg" fill-rule="evenodd" clip-rule="evenodd">
                            <path d="M11 11v-11h1v11h11v1h-11v11h-1v-11h-11v-1h11z"/>
                        </svg>
                        <input multiple type="file" class="hidden" id="upload" on:change={handleFileInput} accept="audio/*"/>
                    </label>
                </div>
                <div class="flex justify-center items-center gap-1 w-[30%] max-lg:hidden">
                    <span class="text-white mr-3 w-[10%]">{currTimeDisplay}</span>
                    <div class="relative w-[80%] rounded-md cursor-pointer h-2">
                        <div class="progress" style="width: {progress}%"></div>
                        <input type="range" class="seek-slider" min={0} max={100} step={1} bind:value={progress} on:input={setProgress}>
                    </div>
                    <span class="text-white w-[10%]">{totalTimeDisplay}</span>
                </div>
                <div class="flex justify-center gap-4 w-[15%]">
                    <button class="button">
                        <svg xmlns="http://www.w3.org/2000/svg" class="svg" viewBox="0 0 24 24">
                            <path d="M2 7h-2v-2h2c3.49 0 5.48 1.221 6.822 2.854-.41.654-.754 1.312-1.055 1.939-1.087-1.643-2.633-2.793-5.767-2.793zm16 10c-3.084 0-4.604-1.147-5.679-2.786-.302.627-.647 1.284-1.06 1.937 1.327 1.629 3.291 2.849 6.739 2.849v3l6-4-6-4v3zm0-10v3l6-4-6-4v3c-5.834 0-7.436 3.482-8.85 6.556-1.343 2.921-2.504 5.444-7.15 5.444h-2v2h2c5.928 0 7.543-3.511 8.968-6.609 1.331-2.893 2.479-5.391 7.032-5.391z"/>
                        </svg>
                    </button>
                    <button on:click={loopAudio} class="button">
                        <svg xmlns="http://www.w3.org/2000/svg" class="svg {isLooped? 'fill-red-700' : ''}" viewBox="0 0 24 24">
                            <path d="M2 12c0 .999.381 1.902.989 2.604l-1.098.732-.587.392c-.814-1.025-1.304-2.318-1.304-3.728 0-3.313 2.687-6 6-6h9v-3l6 4-6 4v-3h-9c-2.206 0-4 1.794-4 4zm20.696-3.728l-.587.392-1.098.732c.608.702.989 1.605.989 2.604 0 2.206-1.795 4-4 4h-9v-3l-6 4 6 4v-3h9c3.313 0 6-2.687 6-6 0-1.41-.489-2.703-1.304-3.728z"/>
                        </svg>
                    </button>
                    <div class="w-1/3 group relative">
                        <button on:click={muteAudio} class="button">
                            <svg xmlns="http://www.w3.org/2000/svg" class="svg" fill-rule="evenodd" clip-rule="evenodd">
                                <path d="{isMuted === true || volume === 0 ? 'M18 23l-9.305-5.998.835-.651 7.47 4.815v-10.65l1-.781v13.265zm0-15.794l5.384-4.206.616.788-23.384 18.264-.616-.788 5.46-4.264h-2.46v-10h5.691l9.309-6v6.206zm-11.26 8.794l1.26-.984v-7.016h-4v8h2.74zm10.26-8.013v-5.153l-8 5.157v6.244l8-6.248z'
                                : 'M15 23l-9.309-6h-5.691v-10h5.691l9.309-6v22zm-9-15.009v8.018l8 5.157v-18.332l-8 5.157zm14.228-4.219c2.327 1.989 3.772 4.942 3.772 8.229 0 3.288-1.445 6.241-3.77 8.229l-.708-.708c2.136-1.791 3.478-4.501 3.478-7.522s-1.342-5.731-3.478-7.522l.706-.706zm-2.929 2.929c1.521 1.257 2.476 3.167 2.476 5.299 0 2.132-.955 4.042-2.476 5.299l-.706-.706c1.331-1.063 2.182-2.729 2.182-4.591 0-1.863-.851-3.529-2.184-4.593l.708-.708zm-12.299 1.299h-4v8h4v-8z'}"/>
                            </svg>
                        </button>
                        <div class="slider hidden group-hover:block absolute left-[-15px] bottom-[-5px]" style="width: {volume * 100}%">
                            <input type="range" min={0} max={1} step={0.01} bind:value={volume} on:input={(e)=>{ audio.volume = e.target.value}}>
                        </div>
                    </div>
                </div>
            </mediabar>
        </homepage>
    </content>
</navbar>