> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.amitmerchant.com](https://www.amitmerchant.com/textarea-auto-increase-height/)

> Textareas areas are great when it comes to accepting a large amount of text from the user. But, the p......

Textareas areas are great when it comes to accepting a large amount of text from the user. But, the problem with textareas is that they have a fixed height. So, if the user enters more text than the height of the text, the text will overflow and the user will have to scroll to see the rest of the text.

You can increase the height of the textarea using the `rows` attribute. But, that’s not a good solution since you don’t know how much text the user will enter. So, you can’t set the `rows` attribute to a fixed value.

Thankfully, an experimental CSS rule with `form-sizing` property is coming that will let you increase the height of the textarea automatically based on the amount of text the user enters.

```
textarea {
    form-sizing: normal;
}


```

Here’s what it looks like in action.

 <video src="" control></video>

The `form-sizing` property is a new CSS rule that’s coming first in Chrome Canary soon. You can check the discussion around this rule [here](https://github.com/w3c/csswg-drafts/issues/7542).

I sincerely hope this rule gets implemented soon. It will be a great addition to the CSS spec.