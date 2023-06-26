# Videoroom - stream receiving management

We must reduce the data sent to each participant to allow a conference to scale. Besides things like simulcast and temporal scalability, a huge reduction might be achieved by selectively stopping unnecessary streams, e.g. when they are hidden

## SFU-side

One of the approaches we’ve tried is to make decisions on our SFU. This module was called Display Manager inside the RTC Engine

### Advantages

- No need for frontend communication
- Quicker reaction to voice detection
- Taking bandwidth into the account

### Disadvantages

- The implementation was poor and thus turned off, i.e. not tested for a while
- Without frontend feedback not possible to adapt to the layout

## Client-side

### Advantages

- May turn off not rendered streams
- More flexible - different clients may have other strategies/views

### Disadvantages

- Slower to react
- Harder to include network stats in the decision-making process, this would possibly require some additional events.

## Competition

- Livekit has an API for both client and backend to disable a negotiated stream, adding on top of that adaptiveStream that allows a client to notify the SFU of visibility status and dimensions of the rendered stream. This allows the SFU to decide which simulcast layer to send.
- Janus offers manual management from the client side of the streams + slowlink events suggesting layer switch

## Outtakes
IMO the best solution, in the long run, is a hybrid approach, combining both approaches, more or less like Livekit. However, since our current implementation on the Server-side is not usable, it’s best to start with client-side mechanisms as they cover most use-cases and we can build on top of that.

When disabling the stream, we don’t want a renegotiation, thus the RTC Engine must be aware of that.

## Tasks

- Add an option to pause a track to RTC Engine
- Add pausing API to Jellyfish Client API
- Maybe add pausing API to Jellyfish Server API
- [Optional] Move the layer-selection logic from Videoroom JS to Jellyfish API
