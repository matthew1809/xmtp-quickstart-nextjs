# Quickstart App

XMTP (Extensible Message Transport Protocol) is an open protocol and network for secure and private web3 messaging. For example, you can build an app with XMTP to send messages between blockchain accounts, including chat/DMs, alerts, announcements, and more.

### Demo App

This repository demonstrates the implementation of these concepts within a simple chat app.

[GitHub repo](https://github.com/fabriguespe/xmtp-quickstart-js)

```tsx
git clone git@github.com:fabriguespe/xmtp-quickstart-js.git
cd xmtp-quickstart-js
npm install
npm run dev
```

### Learning Objectives:

- Setting up the ConnectWallet button
- Signing in with XMTP
- Loading a conversation
- Sending a message

### Install dependencies

```bash
npm install @xmtp/js @thirdweb-dev/react
```

### Configuring the client

First we need to initialize XMTP client using as signer our wallet connection of choice.

```tsx
import Home from "@/components/Home";
import { ThirdwebProvider } from "@thirdweb-dev/react";

export default function Index() {
  return (
    <ThirdwebProvider activeChain="goerli">
      <Home />
    </ThirdwebProvider>
  );
}
```

### Display connect with XMTP

Now that we have the wrapper we can add a button that will sign our user in with XMTP.

```tsx
{
  isConnected && !isOnNetwork && (
    <div className={styles.xmtp}>
      <ConnectWallet theme="light" />
      <button onClick={initXmtp} className={styles.btnXmtp}>
        Connect to XMTP
      </button>
    </div>
  );
}
```

```tsx
// Function to initialize the XMTP client
const initXmtp = async function () {
  // Create the XMTP client
  const xmtp = await Client.create(signer, { env: "production" });
  //Create or load conversation with Gm bot
  newConversation(xmtp, PEER_ADDRESS);
  // Set the XMTP client in state for later use
  setIsOnNetwork(!!xmtp.address);
  //Set the client in the ref
  clientRef.current = xmtp;
};
```

### Load conversation and messages

Now using our hooks we are going to use the state to listen whan XMTP is connected.

Later we are going to load our conversations and we are going to simulate starting a conversation with one of our bots

```tsx
useEffect(() => {
  if (isOnNetwork && convRef.current) {
    // Function to stream new messages in the conversation
    const streamMessages = async () => {
      const newStream = await convRef.current.streamMessages();
      for await (const msg of newStream) {
        const exists = messages.find((m) => m.id === msg.id);
        if (!exists) {
          setMessages((prevMessages) => {
            const msgsnew = [...prevMessages, msg];
            return msgsnew;
          });
        }
      }
    };
    streamMessages();
  }
}, [messages, isOnNetwork]);
```

### Listen to conversations

In your component initialize the hook to listen to conversations

```tsx
const [history, setHistory] = useState(null);
const { messages } = useMessages(conversation);
// Stream messages
const onMessage = useCallback((message) => {
  setHistory((prevMessages) => {
    const msgsnew = [...prevMessages, message];
    return msgsnew;
  });
}, []);
useStreamMessages(conversation, onMessage);
```
