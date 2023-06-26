# Handling death of child linked to parent's (bin's) pad

Currently, in Membrane parents, we have an implicit assumption, that only a parent can affect the shape links between his children. Children themselves cannot influence the links between each other, only parent can affect them, by using `:spec`, `:remove_children` and `:remove_link` actions.

On the other hand, since bins don’t take a direct part in sending buffers between elements, we have an undefined behavior, when bins returns `:remove_children` action with a child, that is linked to its pads (this situation occurs with :remove_link as well). Currently, it does not cause raising an error, but when:

- bin returns `:remove_children` with a child linked to its pad, this child will be aborted, but `handle_pad_removed` in the bin will not be executed (in Element, that was linked to the removed child, it will not happen as well)
- bin returns `:remove_link` with a child and pad linked to its pad, it will cause calling `handle_pad_removed` in the bin and the child, but not in the Component linked with a bin

## Solution proposal

The simplest solution to this problem would be to just raise an error when bin tries to remove a child, that is linked to its pads.

Beyond that, we could also introduce a new action in children, `:remove_pad`, and a new callback in parents, `handle_child_pad_removed(child, pad, ctx, state)`. When the bin wanted to remove a child, that is linked to its pad (or on crash group down), the bin could return a proper `:remove_pad` action, and then, remove a child, that would be no longer linked to bin pads. On the other hand, returning such action in the child would cause calling `handle_child_pad_removed` in the a parent. Suggested implementation of this callback would be to:

- remove a link between children, if a child a with removed pad was linked to another child
- return `:remove_pad` action, if a child with a removed pad was linked to bin pad

## Disadvantages

Until now, only a parent could decide about the shape of links between his children. After the introducing solution proposed above, parent still would know about all existing links between children, but it wouldn’t be the only subject, that will have ability to impact it. To avoid having unexpected changes in links between children, default implementation of `handle_child_pad_removed` could raise an error.

## Advantages

The big advantage of introducing `:remove_pad` and `c:handle_child_pad_removed/4` would be the ability to spawn crash groups in bins, which is now unavailable.

## Problem to consider

What have to consider, is how to handle a situation, when a bin returns `:remove_pad`, but there is a child, that is connected to it via static pad.

## Alternative

Alternatively, we could introduce only `c:handle_child_pad_removed/4` without `:remove_pad` action and execute new callback in 2 cases:

When bin has a crash group linked with its inner pads and crash group dies, `c:handle_child_pad_removed` in bin’s parent would be triggered.

When bin has `c:handle_child_pad_removed` triggered and child with pad passed to this callback relates to bin’s inner pad, `c:handle_child_pad_removed` in bin’s parent would be triggered as well.

Advantage of this solution is a fact, that it is simpler then the first proposal, but, on the other hand, it left less control to the developer writing Membrane Components
