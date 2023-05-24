### Step 0: Install dependencies

```bash
npm install @xmtp/js
```

### Step 1: Configuring the client

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

### Step 2: Display connect with XMTP

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
const clientOptions = {
  env: "production",
};

// Function to initialize the XMTP client
const initXmtp = async function () {
  // Create the XMTP client
  const xmtp = await Client.create(signer, { env: "production" });
  // Register the codecs. AttachmentCodec is for local attachments (<1MB)
  xmtp.registerCodec(new AttachmentCodec());
  //RemoteAttachmentCodec is for remote attachments (>1MB) using thirdweb storage
  xmtp.registerCodec(new RemoteAttachmentCodec());
  //Create or load conversation with Gm bot
  newConversation(xmtp, PEER_ADDRESS);
  // Set the XMTP client in state for later use
  setIsOnNetwork(!!xmtp.address);
  //Set the client in the ref
  clientRef.current = xmtp;
};
```

### Step 3: Load conversation and messages

Now using our hooks we are going to use the state to listen whan XMTP is connected.

Later we are going to load our conversations and we are going to simulate starting a conversation with one of our bots

```tsx
useEffect(() => {
  async function loadConversation() {
    if (await client?.canMessage(PEER_ADDRESS)) {
      const convv = await startConversation(PEER_ADDRESS, "hi");
      setConversation(convv);
      const history = await convv.messages();
      console.log("history", history.length);
      setHistory(history);
    } else {
      console.log("cant message because is not on the network.");
      //cant message because is not on the network.
    }
  }
  if (!conversation && client) loadConversation();
}, [conversation, client, messages]);
```

### Step 4: Listen to conversations

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

### Step 5 (optional): Save keys

We are going to use a help file to storage our keys and save from re-signing to xmtp each time

```tsx
const ENCODING = "binary";

export const buildLocalStorageKey = (walletAddress: string) =>
  walletAddress ? `xmtp:${"dev"}:keys:${walletAddress}` : "";

export const loadKeys = (walletAddress: string): Uint8Array | null => {
  const val = localStorage.getItem(buildLocalStorageKey(walletAddress));
  return val ? Buffer.from(val, ENCODING) : null;
};

export const storeKeys = (walletAddress: string, keys: Uint8Array) => {
  localStorage.setItem(
    buildLocalStorageKey(walletAddress),
    Buffer.from(keys).toString(ENCODING),
  );
};

export const wipeKeys = (walletAddress: string) => {
  // This will clear the conversation cache + the private keys
  localStorage.removeItem(buildLocalStorageKey(walletAddress));
};
```

Then in our main app we can use them for initiating the client

```tsx
//Initialize XMTP
const initXmtpWithKeys = async () => {
  // create a client using keys returned from getKeys
  //Use signer wallet from ThirdWeb hook `useSigner`
  const address = await signer.getAddress();
  let keys = loadKeys(address);
  if (!keys) {
    keys = await Client.getKeys(signer, {
      ...clientOptions,
      // we don't need to publish the contact here since it
      // will happen when we create the client later
      skipContactPublishing: true,
      // we can skip persistence on the keystore for this short-lived
      // instance
      persistConversations: false,
    });
    storeKeys(address, keys);
  }
  await initialize({ keys, options: clientOptions, signer });
};
```
