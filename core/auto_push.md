# Auto-push

This document contains descriptions of how elements containing pads with flow_control: :auto will manage sending demands and buffers in POC version and final version.

The purpose of the changes described in this document is observation, that if we have element `A` with a pull input pad and element `B` with a push output pad linked with each other, element `A` has to send demands at least as fast as element `B` is sending buffers, to avoid toilet overflow. Hence, if `A` is not linked to any source (or filter) with a pull output pad, `A` has to send demand on all of its pull input pads as fast as buffers arrive. 
Therefore, all input pads of `A` could work in push mode.

## POC version

The POC version does not support all cases supported by the final version but is simpler in implementation.

In the POC version, every element containing pads with `flow_control: :auto` can be in one of the three states: `:push`, `:pull` or `:not_resolved`. This state is called `effective_flow_control`. Every pad also has its `effective_flow_control`. For pads with `flow_control: :manual` it will always be `:pull`, for pads with `flow_control: :push` it will always be `:push`, and pads with `flow_control: :auto` will have this same `effective_flow_control`, as the element they belong to.

Auto pads with `effective_flow_control` equal to `:push` will work, as they would have `flow_control: :push`. Auto pads with `:pull` `effective_flow_control` will behave, as they have been behaving until now.

The mechanism of resolving `effective_flow_control` for elements with auto pads is following: in the beginning, every element is in `:not_resolved` state. Elements will try to resolve its `effective_flow_control` on `handle_playing` and, if it will not succeed, on every new pad added.

On every attempt to resolve `effective_flow_control`, element (let’s call it `A`) will look at its siblings, which send buffers to him via his input auto pads. In at least one of them is linked to `A` via output pad with `:pull` `effective_flow_control`, A will resolve its `effective_flow_control` to `:pull`. If all of these siblings are linked via output pads with `:push` `:effective_flow_control`, `A` will resolve its `effective_flow_control` to `:push`. Otherwise, if there are no such a siblings or all of them are linked to `A` via output pads with `effective_flow_control` equal to `:not_resolved`, `A` will stay in `:not_resolved` state.

In the POC version, after exiting `:not_resolved` state, the `effective_flow_control` of an element will never change. Moreover, if `A` has push `effective_flow_control`, and after resolving it, a new source will be linked to `A` via an output pad working in `:pull` `effective_flow_control`, `A` will raise an error.

## Final version

The major difference between the POC version and the final version, is allowing elements to change their effective_flow_control after the first resolution. There will be no `:not_resolved` `effective_flow_control`. The initial value of `effective_flow_control` for all elements will be `:push`. If element `A` will detect, that one of its siblings, that `A` is linked to via input auto pad, works in a `:pull effective_flow_control`, `A` will enter `:pull` `effective_flow_control` and will start sending auto-demands. On the other hand, if element `A` detects, that all elements linked to it via `A's` auto input pads have output pads with `:push` output pads and there is no need to send any auto-demands, `A` will enter `:push` `effective_flow_control` and will stop sending auto-demands on its auto input pads

