<script>
    import Peer from 'peerjs';
    import {generate} from "random-words";


    const peer = new Peer(generate() + Math.floor(Math.random() * 100));

    let status = "Connecting...";

    let peerId = "";
    let selfId = "";

    let conn;

    let localStream;

    let doVideo = true;

    peer.on('open', (id) => {
        console.log(id);

        selfId = id;

        status = "Ready";
    })

    async function call() {
        console.log(peerId);
        status = "Calling...";

        await navigator.mediaDevices.getUserMedia({
            audio: true,
            video: doVideo
        })
            .then((stream) =>{
                localStream = stream;

                document.getElementById("local").srcObject = localStream;
                document.getElementById("local").play()
            })
            .catch((e)=>{
                alert("Failed to get media devices. " + e)
                status = "Ready";


            })

        if(!localStream) return

        conn = peer.call(peerId, localStream)

        conn.on('stream', (stream) => {
            console.log(stream);

            document.getElementById("remote").srcObject = stream
            document.getElementById("remote").play()

            status = "In call";
        })

    }

    peer.on('call', async (call) => {

        let ans = confirm("Call from " + call.peer + ". Accept?");
        if(!ans) return;

        await navigator.mediaDevices.getUserMedia({
            audio: true,
            video: doVideo
        })
            .then((stream) =>{
                localStream = stream;

                document.getElementById("local").srcObject = localStream;
                document.getElementById("local").play()
            })
            .catch((e)=>{
                alert("Failed to get media devices. " + e)
                status = "Ready";


            })

        if(!localStream) return

        call.answer(localStream);

        call.on('stream', (stream) => {
            console.log(stream);

            document.getElementById("remote").srcObject = stream
            document.getElementById("remote").play()

            status = "In call";
        })

    })

</script>


<div class="w-full h-screen flex flex-col items-center justify-center font-mono p-20 text-2xl bg-black text-white">
    <h1>{status}</h1>
    <h2 class="mb-5 text-xl">Peer id: <span class="font-bold">{selfId}</span></h2>

    {#if status === "Ready"}
        <form on:submit|preventDefault={call}>
            <input class="text-black p-2 rounded" placeholder="Peer ID" type="text" bind:value={peerId} />
            <br>

            <label for="video">Enable video: </label>
            <input  id="video" type="checkbox" bind:checked={doVideo}>

            <br>

            <button type="submit" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded transition">Call</button>
        </form>
    {/if}

    <div class="flex flex-row gap-5 h-fit w-fit">
        <video id="local" muted class="max-h-64 max-w-64"></video>

        <!-- svelte-ignore a11y-media-has-caption -->
        <video id="remote" class="max-h-64 max-w-64"></video>
    </div>
</div>

<style>
    :global(body) {
        background-color: black;
    }
</style>
