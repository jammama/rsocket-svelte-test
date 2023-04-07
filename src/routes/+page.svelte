<h1>Welcome to SvelteKit</h1>
<p>Visit <a href="https://kit.svelte.dev">kit.svelte.dev</a> to read the documentation</p>

<script>
    import SockJS from 'sockjs-client/dist/sockjs';
    import Stomp from 'stomp-websocket';
    import {IdentitySerializer, JsonSerializer, RSocketClient} from "rsocket-core";
    import RSocketWebsocketClient  from 'rsocket-websocket-client';

    class EchoResponder {
        constructor(callback) {
            this.callback = callback;
        }
        fireAndForget(payload) {
            this.callback(payload);
        }
    }
    // Create an instance of a client
    async function createClient() {
        const client = new RSocketClient({
            serializers: {
                data: JsonSerializer,
                metadata: IdentitySerializer
            },
            setup: {
                dataMimeType: 'application/json',
                keepAlive: 1000000, // avoid sending during test
                lifetime: 100000,
                metadataMimeType: 'message/x.rsocket.routing.v0',
            },
            responder: new EchoResponder((payload) =>{
                console.log(payload.data);
            }),
            transport: new RSocketWebsocketClient ({
                url: "ws://localhost:1122/rsocket"
            }),
        }).connect().subscribe({
            onComplete: socket => {
                console.log('connected!', socket)
                rsocket = socket
            }
        });
    }
    let rsocket
    const destination = (endpoint) => String.fromCharCode(endpoint.length) + endpoint

    async function run() {
        rsocket ?? await createClient();
    }
    async function send() {
        console.log("send it")
        rsocket.requestResponse({
            data: {
                msgType: "GREET",
                classRoomId: "siz",
                userType: "STUDENT",
                data: userSeq
            },
            metadata: destination('message')
        }).subscribe()
    }


    async function sendVoid(){
    }


    let stompClient = null;
    let connected = false;
    let userSeq;
    let connect = () => {
        var socket = new SockJS('http://localhost:1122/connect');
        stompClient = Stomp.over(socket);
        stompClient.connect({'classRoomId': '63fdbd35ea8988005d6cf90e', userSeq, 'userName': 'testTeacher'}, function (frame) {
            connected = true;
            message = [];
            console.log('Connected: ' + frame);
            stompClient.subscribe('/greetings', function (greeting) {
                showGreeting(JSON.parse(greeting.body).content);
            });
            stompClient.subscribe(`/user/${userSeq}/msg`, function (greeting) {
                showGreeting(JSON.parse(greeting.body).content);
            });
        });
    }


    function disconnect() {
        if (stompClient !== null) {
            stompClient.disconnect();
        }
        connected = false;
        console.log("Disconnected");``
    }

    let name = '';
    function sendName() {
        stompClient.send("/app/hello", {}, JSON.stringify({name}));
    }
    function sendToClassRoom() {
        stompClient.send("/app/classroom/test/1",
            {},
            JSON.stringify({
                messageType: "DRAW_PATH",
                groupId: "test",
                receiver: "test",
                data: "test"
            }));
    }
    let message = [];
    function showGreeting(msg) {
        message.push(msg);
        message = [...message];
    }


</script>
<body>
    <div id="main-content" class="container">
        <div class="row">
            <div class="col-md-6">
                <form class="form-inline">
                    <div class="form-group">
<!--                        <button id="create" class="btn btn-default"-->
<!--                                disabled={connected ? "disabled": ""} on:click={() => create()}>Create ClassRoom</button>-->
                        <button class="btn btn-default" on:click={() => run()}>rconnect</button>
                        <button class="btn btn-default" on:click={() => send()}>rsend</button>
                        <button class="btn btn-default" on:click={() => sendVoid()}>rsendVoid</button>
                        <label for="connect">WebSocket connection:</label>
                        <button id="connect" class="btn btn-default"
                                        disabled={connected ? "disabled": ""}
                                type="submit" on:click={() => connect()}>Connect</button>
                        <button id="disconnect" class="btn btn-default"
                                        disabled={!connected ? "disabled": ""}
                                type="submit" on:click={()=>disconnect()}>Disconnect
                        </button>
                    </div>
                </form>
            </div>
            <div class="col-md-6">
                <form class="form-inline">
                    <div class="form-group">
                        <label for="name">What is your name?</label>
                        <input type="text" id="name" class="form-control" placeholder="test userSeq..." bind:value={userSeq}>
                    </div>
                    <button id="send" class="btn btn-default" type="submit" on:click={()=>sendName()}>Send</button>
                    <button on:click={()=>sendToClassRoom()}>Send to class room</button>
                </form>
            </div>
        </div>
        <div class="row">
            <div class="col-md-12">
                {#if connected}
                    <table id="conversation" class="table table-striped">
                        <thead>
                        <tr>
                            <th>Greetings</th>
                        </tr>
                        </thead>
                        <tbody id="greetings">
                        {#each message as msg}
                            <tr><td>{msg}</td></tr>
                        {/each}
                        </tbody>
                    </table>
                {/if}
            </div>
        </div>
    </div>
</body>
