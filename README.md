# useWhisper()

React Hook for OpenAI Whisper API with speech recorder and silence removal built-in

---

_Try OpenAI API price calculator, token counter, and dataset manager (preview)_

[https://openai-price-calculator.web.app](https://openai-price-calculator.web.app)

- ### Install

`npm i @chengsokdara/use-whisper`

`yarn add @chengsokdara/use-whisper`

- ### Usage

- ###### Provide your own OpenAI API key

```jsx
import { useWhisper } from '@chengsokdara/use-whisper'

const App = () => {
  const {
    recording,
    speaking,
    transcribing,
    transcript,
    pauseRecording,
    startRecording,
    stopRecording,
  } = useWhisper({
    apiKey: env.process.OPENAI_API_TOKEN, // YOUR_OPEN_AI_TOKEN
  })

  return (
    <div>
      <p>Recording: {recording}</p>
      <p>Speaking: {speaking}</p>
      <p>Transcribing: {transcribing}</p>
      <p>Transcribed Text: {transcript.text}</p>
      <button onClick={() => startRecording()}>Start</button>
      <button onClick={() => pauseRecording()}>Pause</button>
      <button onClick={() => stopRecording()}>Stop</button>
    </div>
  )
}
```

_**NOTE:** by providing apiKey, it could be exposed in the browser devtool network tab_

- ###### Custom REST API (if you want to keep your OpenAI API key secure)

```jsx
import { useWhisper } from '@chengsokdara/use-whisper'

const App = () => {
  /**
   * you have more control like this
   * do whatever you want with the recorded speech
   * send it to your own custom server
   * and return the response back to useWhisper
   */
  const onTranscribe = (blob: Blob) => {
    const base64 = await new Promise<string | ArrayBuffer | null>(
      (resolve) => {
        const reader = new FileReader()
        reader.onloadend = () => resolve(reader.result)
        reader.readAsDataURL(blob)
      }
    )
    const body = JSON.stringify({ file: base64, model: 'whisper-1' })
    const headers = { 'Content-Type': 'application/json' }
    const { default: axios } = await import('axios')
    const response = await axios.post('/api/whisper', body, {
      headers,
    })
    const { text } = await response.data
    // you must return result from your server in Transcript format
    return {
      blob,
      text,
    }
  }

  const { transcript } = useWhisper({
    // callback to handle transcription with custom server
    onTranscribe,
  })

  return (
    <div>
      <p>{transcript.text}</p>
    </div>
  )
}
```

- ### Examples

- ###### Remove silence before sending to Whisper to save cost

```jsx
import { useWhisper } from '@chengsokdara/use-whisper'

const App = () => {
  const { transcript } = useWhisper({
    apiKey: env.process.OPENAI_API_TOKEN, // YOUR_OPEN_AI_TOKEN
    // use ffmpeg-wasp to remove silence from recorded speech
    removeSilence: true,
  })

  return (
    <div>
      <p>{transcript.text}</p>
    </div>
  )
}
```

- ###### Auto start recording on component mounted

```jsx
import { useWhisper } from '@chengsokdara/use-whisper'

const App = () => {
  const { transcript } = useWhisper({
    // will auto start recording speech upon component mounted
    autoStart: true,
  })

  return (
    <div>
      <p>{transcript.text}</p>
    </div>
  )
}
```

- ###### Keep recording as long as the user is speaking

```jsx
import { useWhisper } from '@chengsokdara/use-whisper'

const App = () => {
  const { transcript } = useWhisper({
    apiKey: env.process.OPENAI_API_TOKEN, // YOUR_OPEN_AI_TOKEN
    nonStop: true, // keep recording as long as the user is speaking
    stopTimeout: 5000, // auto stop after 5 seconds
  })

  return (
    <div>
      <p>{transcript.text}</p>
    </div>
  )
}
```

- ###### Auto transcribe speech when recorder stopped

```jsx
import { useWhisper } from '@chengsokdara/use-whisper'

const App = () => {
  const { transcript } = useWhisper({
    autoTranscribe: true, // will try to automatically transcribe speech
  })

  return (
    <div>
      <p>{transcript.text}</p>
    </div>
  )
}
```

- ###### Customize Whisper API config when autoTranscribe is true

```jsx
import { useWhisper } from '@chengsokdara/use-whisper'

const App = () => {
  const { transcript } = useWhisper({
    apiKey: env.process.OPENAI_API_TOKEN, // YOUR_OPEN_AI_TOKEN
    whisperConfig: {
      prompt: 'previous conversation', // you can pass previous conversation for context
      response_format: 'text', // output text instead of json
      temperature: 0.8, // random output
      language: 'es', // Spanish
    },
  })

  return (
    <div>
      <p>{transcript.text}</p>
    </div>
  )
}
```

- ### Dependencies

  - **@chengsokdara/react-hooks-async** asynchronous react hooks
  - **recordrtc:** cross-browser audio recorder
  - **@ffmpeg/ffmpeg:** for silence removal feature
  - **hark:** for speaking detection
  - **axios:** since fetch does not work with Whisper endpoint

_most of these dependecies are lazy loaded, so it is only imported when it is needed_

- ### API

- ###### Config Object

| Name           | Type                                               | Default Value | Description                                                                                                          |
| -------------- | -------------------------------------------------- | ------------- | -------------------------------------------------------------------------------------------------------------------- |
| apiKey         | string                                             | ''            | your OpenAI API token                                                                                                |
| autoStart      | boolean                                            | false         | auto start speech recording on component mount                                                                       |
| autoTranscribe | boolean                                            | false         | should auto transcribe after stop recording                                                                          |
| nonStop        | boolean                                            | false         | if true, record will auto stop after stopTimeout. However if user keep on speaking, the recorder will keep recording |
| removeSilence  | boolean                                            | false         | remove silence before sending file to OpenAI API                                                                     |
| stopTimeout    | number                                             | 5,000 ms      | if nonStop is true, this become required. This control when the recorder auto stop                                   |
| whisperConfig  | [WhisperApiConfig](#whisperapiconfig)              | undefined     | Whisper API transcription config                                                                                     |
| onTranscribe   | (blob: Blob) => Promise<[Transcript](#transcript)> | undefined     | callback function to handle transcription on your own custom server                                                  |

- ###### WhisperApiConfig

| Name            | Type   | Default Value | Description                                                                                                                                                                                                                                                                                                                                               |
| --------------- | ------ | ------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| prompt          | string | undefined     | An optional text to guide the model's style or continue a previous audio segment. The prompt should match the audio language.                                                                                                                                                                                                                             |
| response_format | string | json          | The format of the transcript output, in one of these options: json, text, srt, verbose_json, or vtt.                                                                                                                                                                                                                                                      |
| temperature     | number | 0             | The sampling temperature, between 0 and 1. Higher values like 0.8 will make the output more random, while lower values like 0.2 will make it more focused and deterministic. If set to 0, the model will use [log probability](https://en.wikipedia.org/wiki/Log_probability) to automatically increase the temperature until certain thresholds are hit. |
| language        | string | en            | The language of the input audio. Supplying the input language in [ISO-639-1](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) format will improve accuracy and latency.                                                                                                                                                                             |

- ###### Return Object

| Name           | Type                      | Description                                                               |
| -------------- | ------------------------- | ------------------------------------------------------------------------- |
| recording      | boolean                   | speech recording state                                                    |
| speaking       | boolean                   | detect when user is speaking                                              |
| transcribing   | boolean                   | while removing silence from speech and send request to OpenAI Whisper API |
| transcript     | [Transcript](#transcript) | object return after Whisper transcription complete                        |
| pauseRecording | Promise                   | pause speech recording                                                    |
| startRecording | Promise                   | start speech recording                                                    |
| stopRecording  | Promise                   | stop speech recording                                                     |

- ###### Transcript

| Name | Type   | Description                                |
| ---- | ------ | ------------------------------------------ |
| blob | Blob   | recorded speech in JavaScript Blob         |
| text | string | transcribed text returned from Whisper API |

- ### Roadmap

  - react-native support, will be export as use-whisper/native

---

**_Contact me for web or mobile app development using React or React Native_**  
[https://chengsokdara.github.io](https://chengsokdara.github.io)
