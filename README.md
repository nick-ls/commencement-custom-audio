# Commencement 2024
It's time for UCSD Commencement! 

This commencement, however, is not like the others. It comes with a *sinister twist* 😨.

![image](https://github.com/nick-ls/commencement-custom-audio/assets/22565232/3223549f-4b8e-4edf-9fc4-ab8dcb63e45c)

They are using **AI** to read out people's names for a more:
<ul style="list-style:none">
<h3>🏃💨 <em>efficient</em></h3>
<h3>✨<em>streamlined</em>✨</h3>
<h3>and <em><a href="https://www.youtube.com/watch?v=KfnQhlBqYwM">"accurate"</a></em> 🤓☝️</h3>
</ul>
graduation experience.
<br/>
<br/>
Linguists weep as yet another job is automated away by AI.

<br/>
<br/>

> [!IMPORTANT]
> <table><tbody><tr><td><img height="60px" src="https://github.com/nick-ls/commencement-custom-audio/assets/22565232/5b74b96e-2595-400b-8f86-5f82e30d61f5"></td> <td><h2><a href="#how-to">SKIP TO THE TUTORIAL</a> if you're in a rush or have TikTok brain</h2></td></tr></tbody></table>

# Discovery

Thank you to [@sheeptester](https://sheeptester.github.io/) and ⊆ for informing me that all this is possible!

Let's dive into the UI:

![image](https://github.com/nick-ls/commencement-custom-audio/assets/22565232/6f9f91a9-563d-4d8c-a782-0e0a1533aa5e)


It's a fairly simple 4-step process
1) Enter your name
2) Choose a voice (unfortunately there are only 2 to choose from... [or is there?](#baked-in-api-key))
3) Load your pronunciation sample
    1) From there you can listen to the AI mess up your name pronunciation and give a disappointed sigh. 
4) Approve your name recording

So, what exactly is going on when you follow these steps?

Choosing your name and a voice doesn't do anything besides send session keep-alive requests, but something interesting happens when you click "Load Pronunciation Sample"

![image](https://github.com/nick-ls/commencement-custom-audio/assets/22565232/a97074e6-a5ba-4fec-aedb-e4ecf3f36494)

It queries an API to get an audio file, which is what is added to the page for you to listen to! That's a start.

Examining the HTML in devtools, we can see that the audio file we generated in step 3 is added to the page as a blob

![image](https://github.com/nick-ls/commencement-custom-audio/assets/22565232/bd366654-37a3-4d99-9808-d3a2df90e8c2)

Assuming that this site was securely designed, from this point we would expect that clicking "Approve Name Recording" would mark the recording we generated in step 3 as an accepted name recording. Nothing extra would be sent to the server besides this confirmation, and the generated audio file would already exist in some database or filesystem on that server waiting for your approval. Let's see what actually happens.

![image](https://github.com/nick-ls/commencement-custom-audio/assets/22565232/1a3410b7-0db1-4eb9-ab26-cf095c62da5e)

A request is sent to the `/frontend/registrant/upload-file` endpoint! That's curious! What possible reason could we have for uploading a file to the server? Let's look a little closer to see what this file is that we're sending:

![image](https://github.com/nick-ls/commencement-custom-audio/assets/22565232/2af3420c-fd4e-4662-bfff-0858600386ab)

Hmm, seems an awful lot like we're uploading an audio file to their server. Considering that the filename is `voice.mp3` and that the file header starts with `ID3` (which Wikipedia says "is a metadata container most often used in conjunction with the MP3 audio file format"), I would say that we're uploading a client-supplied file to the server! This is exactly what we want!

Now how do we supply our own file to this endpoint in a way it can understand? We could copy the request and resend it with a different file, but there's definitely a simpler way to do it using our old friend inspect element and devtools!

![image](https://github.com/nick-ls/commencement-custom-audio/assets/22565232/fd203558-cd30-4503-8022-4581052f744a)

When we clicked the "APPROVE Name Recording" button, the UI changed and displayed this green progress bar (almost as if it were uploading a file). Let's look a little closer at it to see what's going on.

![image](https://github.com/nick-ls/commencement-custom-audio/assets/22565232/982a59fd-c5b7-4cfb-be0b-88a890e1cc72)


Wow that's a lot of hidden elements! There's plenty of `display: none !important`s and `type="hidden"`s all over the place here. Looks like there's something here that someone didn't want people finding. Let's fix up those hidden styles real quick...

![image](https://github.com/nick-ls/commencement-custom-audio/assets/22565232/a853460d-8ade-4443-85b6-d7e2a413544b)

Who could have known that there was all of this hiding underneath all of that pesky CSS? Looks like someone put an entire file upload input on the page for us to submit our custom audio file! How kind of them 🥰

Let's click remove on the file and put one of our own in there.

![image](https://github.com/nick-ls/commencement-custom-audio/assets/22565232/ad9cb170-10b1-4a36-a4b6-62476bd917c8)

Ah much better! Now all that's left to do is approve our name recording!

![image](https://github.com/nick-ls/commencement-custom-audio/assets/22565232/c9a59d1a-32a4-4b03-af67-da0b5e486393)

Click!

![image](https://github.com/nick-ls/commencement-custom-audio/assets/22565232/37675620-e95b-47fc-a6e8-617c99776898)

Click!

![image](https://github.com/nick-ls/commencement-custom-audio/assets/22565232/95d181c3-6330-463d-b852-5d89f3f57bb1)

If all went according to plan, this should be the audio file you uploaded to the input!

Sometimes there's a bug where if you generated a name sample already, you'll softlock yourself into not being able to upload your own file, but if that happens, just do this again but don't generate a name file. Alternatively [follow the more efficient steps below](#how-to)

# How to
So you've decided that you don't want to have an AI voice mispronounce your name? Great choice!

### Step 1: Enter your name

![image](https://github.com/nick-ls/commencement-custom-audio/assets/22565232/685f9a21-a469-4b80-be85-8385929a36f4)

### Step 2: Editing the HTML
DO NOT click `Load Pronunciation Sample` or you will be so sad.

![image](https://github.com/nick-ls/commencement-custom-audio/assets/22565232/b189f0e8-498e-4bb8-93cd-46ae2b765e9b)

Instead, right click on the `🗑️Remove` icon and select `Inspect (Q)` or `Inspect element` if you use a browser with ads. 

Find the HTML elements that look like the elements below. You can also `Ctrl`+`F` for `Browse …` (copy-and-paste this exactly) while in the devtools inspect panel to find the elements below if you're having trouble. 

![image](https://github.com/nick-ls/commencement-custom-audio/assets/22565232/e9590c9f-bbe6-420d-8d87-4a79bd084d36)

Now select the `<div class="btn btn-default btn-file" tabindex="-1">` element and scroll in the style browser until you find the style for `.btn-file` and uncheck the box next to `display: none !important` or delete it in some other way

![image](https://github.com/nick-ls/commencement-custom-audio/assets/22565232/f14f8b7f-cba2-48e0-bd15-503c564ff07a)

You can optionally remove any `display: none` properties to the cancel button, but it is not necessary to do so. If you have done the steps correctly so far, looking back at the page should show you a `🗑️Remove` button as well as a `📂Browse` button.

![image](https://github.com/nick-ls/commencement-custom-audio/assets/22565232/ece2e0dc-614e-4253-be97-05234385e68a)

### Step 3: Uploading your file

Click browse and your file manager of choice will open up, allowing you to select any file! I would recommend only submitting `.mp3` files of your name to show UCSD that students can be trusted to upload their own audio, but I'm not your mom, do what you want.

Once you've selected your file to upload, you should see a green progress bar appear that says `Done`

![image](https://github.com/nick-ls/commencement-custom-audio/assets/22565232/7e367a8c-6615-4bc3-8af0-d0ff72b313d8)

At this point, you should click `APPROVE Name Recording`

![image](https://github.com/nick-ls/commencement-custom-audio/assets/22565232/1054ae7e-1264-478f-983f-7c43435b53bd)

And fill out any extra things that are required on the page like your Phonetic Full Name. 

Click `Continue➡️` to save your changes.

![image](https://github.com/nick-ls/commencement-custom-audio/assets/22565232/d08ef5b0-0dfc-4e2e-99c9-40f825ec8075)

### Step 4: Revel in the glory that is client-side verification

Congrats grad, you have done it! You've beat client-side verification! I'm so proud of you 🥰

![image](https://github.com/nick-ls/commencement-custom-audio/assets/22565232/d17e3721-317e-4ed0-8ebf-3e26d99cdeca)

Listening to this audio file should play the audio file you selected in step 3. If it isn't then you're cooked 👨‍🍳

# Other fun things
### Baked-in API key 
![image](https://github.com/nick-ls/commencement-custom-audio/assets/22565232/17620916-1340-4611-adce-12e455c4d34c)

In their efforts to make commencement as efficient as possible, it seems that they have exposed their API key for [WellSaidLabs](https://wellsaidlabs.com/) to every logged in client. This effectively gives anyone with the key access to all of the voices offered by WellSaidLabs for no cost to them! After checking with other students, this API key is the same across multiple different logged-in clients, which means that just about anyone could have unrestricted access to their ["Creative"](https://wellsaidlabs.com/pricing/) tier (assumption from number of voices the API key has access to), which boasts a total of 80+ voice styles!
Unfortunately none of them are very good. 

![image](https://github.com/nick-ls/commencement-custom-audio/assets/22565232/b87019cc-c408-4227-9e84-4b82b4977d22)


### Alternatives to AI
Since AI is buggy and prone to mispronouncing peoples' names, I propose that we use [DECtalk](https://en.wikipedia.org/wiki/DECtalk) as a good alternative. DECtalk allows for explicit pronunciation of different phonemes, as well as pitch modulation and different durations per phoneme. 
<table >
    <tbody>
        <tr>
            <td valign="top">
                <img src="https://github.com/nick-ls/commencement-custom-audio/assets/22565232/d8394a89-6171-4529-a7af-498fdf250567">
            </td>
            <td valign="top">
                <img src="https://github.com/nick-ls/commencement-custom-audio/assets/22565232/65cb384b-18d1-4d58-b211-0e65cfbfbc5b">
            </td>
        </tr>
    </tbody>
</table>

Additionally DECtalk has stood the test of time. AI is still a new and evolving technology, while DECtalk has been around as early as 1983! With over 40 years of experience, I would trust DECtalk far more than this newfangled "artificial intelligence" 🙄. I prefer my voice dictation software to be *natural* thank you very much. 


[You can try it online here!](https://tts.cyzon.us/)
