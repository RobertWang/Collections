> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [boostpixels.com](https://boostpixels.com/guide)

> Many beginners make the mistake of typing a single word and hoping for a miracle. Despite impressive ......

Many beginners make the mistake of typing a single word and hoping for a miracle. Despite impressive advances in AI models, mind-reading is still beyond our reach.

To achieve satisfying outputs, it's crucial to understand how to write effective prompts.

Understanding the Base Model
----------------------------

Consider the below input photo to train your model in BoostPixels.

![](https://boostpixels.com/sites/default/files/inline-images/mtpdnx_0.jpg)

Upon completion of the training process, providing an empty prompt to the BoostPixels model generates an image that closely resembles the input photograph used during the training phase.

![](https://boostpixels.com/sites/default/files/inline-images/empty-prompt.jpg)Prompt "" (empty)

This remarkable accomplishment indicates that our 512x512 image has been successfully encoded into the AI model's latent space, which doesn't store pixel data but relies on weights within its structure. The underlying AI model, which is Stable Diffusion, involves a diffusion process for encoding and decoding data.

In diffusion AI models, data is converted into points or vectors by introducing noise to the original data and learning to remove it. The AI model features a network of interconnected nodes with connections of varying strengths, known as "weights." By adjusting these weights, the AI model identifies patterns in the data, enabling it to transform the points in the latent space back into high-dimensional images using its acquired knowledge.

It is also worth noting that the default word the input is trained on is "**ftpdnx**". This word is already included when selecting the model, so there's no need to input it manually every time in your prompt. However, repeating this word can provide additional weight, as it influences the AI's output. The process of adjusting weights to guide the AI's output will be explained further in this guide.

Word Weight
-----------

The most frequent error is typing a single word, such as "bald," and expecting the model to comprehend and produce an image of a bald person.

![](https://boostpixels.com/sites/default/files/inline-images/bald_0.jpg)Prompt: "bald"

This is where people often get disappointed, questioning if these shortcomings are specific to BoostPixels or the current state of AI advancement. The key issue is that users aren't specific enough about their desired output. In prompt writing, understanding the"weight" concept and being explicit is crucial for better results.

It's essential to structure the prompt so that the most critical words appear at the beginning. Each subsequent word carries less weight than the previous one. To give more weight to a word later in the prompt, you can use parentheses"()" to emphasize its importance. This approach guides the noise elimination process, ensuring that the model focuses on the key aspects of the prompt.

Imagine distributing a total of 100 points among words in a prompt. We can illustrate it like this:

Thus, the generated image will primarily reflect the original trained input word, along with the supplementary instructions we provide. However, this may not achieve the desired balance for generating a bald person image.

By enclosing a word in parentheses "()", we can alter the weight it receives, as demonstrated below:

In most cases, this adjustment may still be insufficient to generate an image of an actual bald person. To further modify the balance, we can add "((()))" additional parentheses around the word, like this:

![](https://boostpixels.com/sites/default/files/inline-images/bald-weight.jpg)Prompt: "(((bald)))"

However, simply adding all weight to a single word isn't effective, as it causes the model to focus solely on that word, ignoring the input image.

If we use 6 parentheses "(((((())))))", the weight distribution will look like this:

![](https://boostpixels.com/sites/default/files/inline-images/bald-too-much-weight.jpg)Prompt: "((((((bald))))))"

When unbalanced emphasis is placed on a specific term during input processing, the model tends to disproportionately focus on that term. Consequently, the resulting generated images may exhibit anomalous or unexpected features, deviating from the intended output.

The initial weights assigned to words can vary depending on their frequency in the training dataset. High-frequency words possess greater influence due to their prevalence, and thus, it is essential to carefully calibrate the significance of distinct words in your input to ensure an accurate representation of the intended meaning.

If the generated image does not resemble the desired subject, consider appending the token "(ftpdnx)" to your prompt. This token signals to the AI model that it should reference the specific photograph utilized during its training phase, thereby improving the model's ability to generate a more accurate image.

Utilizing repetition of a keyword rather than parentheses in a prompt can provide comparable guidance to an AI model. For instance, the prompt "bald bald bald bald bald bald bald bald" conveys a similar emphasis as the parenthesized version "((bald))". However, this repetition-based approach may be considered less preferable, as it demands greater effort to discern the weight assigned to a word and is less convenient for adjustment.

Another option, rather than using parentheses in the prompt, is to include synonyms and extra descriptions to steer the model towards generating the desired output image. For instance, using a prompt like "bald hairless baldheaded shaven shaved glabrous" can achieve comparable or even superior results.

Under the hood, this approach modifies the weight distribution, enabling a more effective influence on the output image.

<table><tbody><tr><td>ftpdnx</td><td>person</td><td>bald</td><td>hairless</td><td>baldheaded</td><td>shaven</td><td>shaved</td><td>glabrous</td></tr><tr><td>40</td><td>20</td><td>12</td><td>9</td><td>7</td><td>6</td><td>4</td><td>2</td></tr></tbody></table>

So the words to influence the output image have still a total of 40.

Beyond Universal Prompts: Harnessing Contextual Awareness 
----------------------------------------------------------

In the world of AI image generation, it is crucial to recognize that there is no one-size-fits-all formula that can cater to every individual's specific needs and desired outcomes. The sheer diversity of use cases and objectives demands a more nuanced, context-dependent approach when it comes to crafting prompts and applying techniques.

Each person's goals and expectations for their desired output image are shaped by their unique personal context. As such, it is essential to consider the specific context when determining the appropriate balance and distribution of weights in the image generation process.

For instance, some users may only be interested in making minor adjustments to their input image, such as adding or removing hair while preserving the original photo's visual style. In these cases, leveraging a powerful tool like BoostPixels can lead to impressive results, enabling image manipulation that surpasses the capabilities of even the most skilled professional image editors.

On the other hand, there are those who seek to transform the medium and style of their original image entirely. To accommodate these varying objectives, it is important to explore and experiment with various prompts, specifically tailored to the desired outcome.

Medium and Style
----------------

Specifying medium and style is crucial in image generation. If the AI lacks specific information about the desired style and medium, it defaults to the style of the input image. To optimize outcomes, it is advisable to provide a well-chosen style and medium, as they considerably impact the resulting image corresponding to the subject.

Examples of some commonly used styles and medium:

Styles:

1.  Hyperrealistic
2.  Surrealistic
3.  Cartoon
4.  Watercolour
5.  Sketch
6.  Concept art

Mediums:

1.  Photography
2.  Illustration
3.  Painting
4.  3D rendering

![](https://boostpixels.com/sites/default/files/inline-images/bald-watercolour-painting.jpg)Prompt: "(((bald))) watercolour painting"

Think about the weight distribution of the words in this prompt.

<table><tbody><tr><td>ftpdnx</td><td>person</td><td>bald</td><td>watercolour</td><td>painting</td></tr><tr><td>35</td><td>15</td><td>35</td><td>10</td><td>5</td></tr></tbody></table>

Adding an Artist
----------------

Incorporating an artist's name in your prompt can effectively generate images that appeal to a broad audience. For instance, to create an image with a particular style, include"painting by Rembrandt" in the prompt.

![](https://boostpixels.com/sites/default/files/inline-images/image_12.png)Prompt: "((bald)) painting by Rembrandt"

I've selected Rembrandt as an example to vividly demonstrate the application of a famous artist's style. You can incorporate the names of various renowned artists to adopt their distinctive styles. You can even experiment by merging the styles of multiple artists, giving birth to a truly unique and captivating visual experience.

Here are some examples of popular artist styles combined to create a unique visual style:

Quality Attributes
------------------

It is important to consider quality attributes when writing prompts as they have a direct impact on the quality of the generated outputs. A well-written prompt with strong quality attributes can increase the likelihood of producing desired outputs because it tells the model to guide the noise elimination process involving also these weights. 

Resolution:

1.  1080 HD
2.  4K UHD
3.  8K UHD

Details:

1.  Detailed
2.  Sharp Focus

Lighting:

1.  Natural light
2.  Fluorescent light
3.  LED light
4.  Incandescent light

![](https://boostpixels.com/sites/default/files/inline-images/hyperrealistic.jpg)Prompt: "(((bald))) photo Hyperrealistic 4K UHD Sharp Focus LED light"

Context
-------

Now comes the fun part. Because the intention is often to place the subject also in a different context. This creates often the desired funny or interesting images.

Here are some examples of placing a person in Venice:

![](https://boostpixels.com/sites/default/files/inline-images/venice.jpg)Prompt: "((portrait)) photo hyperrealistic 4K UHD sharp focus LED light intricate Venice background"

Here are some examples of placing a person in Paris:

![](https://boostpixels.com/sites/default/files/inline-images/paris.jpg)Prompt: "((portrait)) photo hyperrealistic 4K UHD sharp focus LED light intricate Paris background"

Of course, context is not limited to the background. Another powerful concept is the "as" construction.

![](https://boostpixels.com/sites/default/files/inline-images/santa.jpg)Prompt: "as (santa claus) photo hyperrealistic 4K UHD sharp focus LED light intricate background"

Here are some examples of representing a person as a superhero:

![](https://boostpixels.com/sites/default/files/inline-images/superhero.jpg)Prompt: "as (super hero) photo hyperrealistic 4K UHD sharp focus LED light intricate background"

Here are some examples of representing the subject wearing a puffy jacket:

![](https://boostpixels.com/sites/default/files/inline-images/puffy-jack.jpg)Prompt: "wearing a ((white puffy)) fashion jacket full body shot photo hyperrealistic 4K UHD sharp focus LED light streetscape, polycount, antipodeans, maximalist, ultra hd, stylish"

The AI model lacks any understanding of Boolean logic. Therefore, writing phrases like "a person riding a horse, beneath an elephant" won't be processed logically. Keep in mind that the words in prompts simply influence the weights in latent space, without any logical, rational or social concepts.

Color
-----

Incorporating a color in a prompt generally affects the entire image. This occurs because the AI model often interprets the specified color as a dominant theme, resulting in the color being applied broadly across the image. Isolating the color to a specific object within the image can be quite challenging, as the model's understanding of the prompt may not be precise enough to localize the color application accurately.

Emotions
--------

Emotional words play a vital role in influencing the image generation process, as they help convey the intended mood or atmosphere of the image.

1.  Positive Emotions: Use words that evoke uplifting or pleasant feelings. Examples: joyful, cheerful, peaceful, enchanting, or vibrant.
2.  Negative Emotions: Use words that convey darker or more somber moods. Examples: melancholic, eerie, ominous, gloomy, or unsettling.
3.  Calm Emotions: Use words that suggest a sense of tranquility or serenity. Examples: serene, tranquil, soothing, or calming.
4.  Energetic or Dynamic Emotions: Use words that imply movement, activity, or excitement. Examples: bustling, lively, dynamic, or energetic.
5.  Mysterious or Enigmatic Emotions: Use words that hint at an air of mystery or intrigue. Examples: mysterious, intricate, enigmatic, cryptic, or mystic.

Perspective and Distance
------------------------

When prompting the AI to generate an image, it's important to specify the desired proximity and distance. For a full-body depiction, use terms like"jacket,""belt," and "shoes" to capture the entire figure.

Incorporating keywords related to the intended style, such as renowned photographers' names, can also help convey the desired distance and perspective.

![](https://boostpixels.com/sites/default/files/inline-images/image_21.png)Prompt: "wearing a brown leather jacket and belt and shoes, forest background, in the style of Brandon Stanton, Humans of New York"

Faces
-----

Generating accurate faces of far-distant persons is currently one of the most challenging tasks for generative AI models. While it is relatively easy to generate satisfying images that are similar to the reference image in terms of distance, it is very hard to generate accurate faces of a person who is far away.

However, there is an effective way to achieve this. The trick is to generate the image in a higher resolution.

Under the hood, it is applying the high resolution fix that helps to correct faces. BoostPixels also applies the upscale algorithm Real-ESRGAN to improve images making them look sharper, clearer, and more visually appealing.

"Enhance face" uses CodeFormer to generate high-quality, natural-looking faces to obtain visually appealing results.

The disadvantage is that generating a high-resolution image takes considerably longer. At the moment, generating a high-resolution 1024 px image takes around 8 seconds, compared to 2 seconds for a 512 px image. This is also the reason that it costs 4 credits.

<table><tbody><tr><td>1024 pixels + Enhace face</td><td><img class="" src="https://boostpixels.com/sites/default/files/inline-images/1024%2Bfe.jpg">Prompt: "detailed photograph of a man wearing a old grey leather jacket and belt, waist shot, forest background, in the style of Brandon Stanton, Humans of New York"</td></tr><tr><td>1024 pixels</td><td><img class="" src="https://boostpixels.com/sites/default/files/inline-images/1024.jpg">Prompt: "detailed photograph of a man wearing a old grey leather jacket and belt, waist shot, forest background, in the style of Brandon Stanton, Humans of New York"</td></tr><tr><td>512 pixels</td><td><img class="" src="https://boostpixels.com/sites/default/files/inline-images/512.jpg">Prompt: "detailed photograph of a man wearing a old grey leather jacket and belt, waist shot, forest background, in the style of Brandon Stanton, Humans of New York"</td></tr></tbody></table>

It is clear that generating at 1024 pixels with Enhance face is adding much more accuracy to the face, which is desperately needed.

Negative prompts
----------------

By default, BoostPixels uses the negative prompt "cartoon 3d hands deformed ugly" to assist beginners in easily achieving satisfactory results. Users can opt for an alternative or input their own custom negative prompt.

It's important to note that negative prompts don't directly remove an object from a generated image. Instead, they assign negative weight to certain words, making positive words more prominent in comparison. It's better to think in terms of points and vectors in latent space, rather than treating the prompt like a 3D model or a pixel-based raster image description.

<table><tbody><tr><td>No negative prompt</td><td><img class="" src="https://boostpixels.com/sites/default/files/inline-images/image.png">Prompt: "digital illustration of a man wearing a leather jacket, Victorian aesthetics, waist shot, forest background, in the style of Magali Villeneuve"</td></tr><tr><td>cartoon 3d hands deformed ugly</td><td><img class="" src="https://boostpixels.com/sites/default/files/inline-images/image_0.png">Prompt: "digital illustration of a man wearing a leather jacket, Victorian aesthetics, waist shot, forest background, in the style of Magali Villeneuve"</td></tr><tr><td>lowres, text, error, cropped, worst quality, low quality, jpeg artifacts, ugly, duplicate, morbid, mutilated, out of frame, extra fingers, mutated hands, poorly drawn hands, poorly drawn face, mutation, deformed, blurry, dehydrated, bad anatomy</td><td><img class="" src="https://boostpixels.com/sites/default/files/inline-images/image_1.png">Prompt: "digital illustration of a man wearing a leather jacket, Victorian aesthetics, waist shot, forest background, in the style of Magali Villeneuve"</td></tr></tbody></table>

A practical approach to understanding the influence of negative prompts is to generate an image using the negative prompt itself. By doing this, you can observe the content that the model is likely being guided away from when generating images based on your original prompt.

Power of repetition and relentlessly trying again and again 
------------------------------------------------------------

Generating the perfect image often requires some trial and error, so don't be disheartened if your first attempt doesn't quite match your vision. It's common to create multiple images and select the best one. Remember, you're not alone or lacking in skill if your prompt doesn't consistently produce the desired results.

This variability is inherent in the diffusion AI model, where outcomes are heavily influenced by random noise. Just like life itself, sometimes you strike gold, and other times you need a bit more persistence. Embrace the process and enjoy the journey, knowing that each iteration increases your chance to generate that ideal image you're imagining.

Sometimes, it can be more effective to repeatedly generate images using the existing prompt, essentially "rolling the dice" many times, instead of trying to perfect the prompt. This approach relies on brute force and a certain degree of luck to produce the desired image, rather than refining the prompt.

This idea is reminiscent of the "infinite monkey theorem," which posits that given infinite time, a monkey randomly hitting keys on a typewriter would eventually produce the complete works of Shakespeare. Similarly, by generating a large number of images using the same prompt, there's a chance you'll eventually obtain the desired output. However, it's important to balance the reliance on brute force and luck with thoughtful prompt design to achieve optimal results in a reasonable amount of time.

Length of prompts
-----------------

BoostPixels sets a limit of 256 characters for prompts, even though its underlying AI model can handle up to 380 characters. This constraint is designed to encourage users to create concise and effective prompts, as longer prompts don't necessarily lead to better results.

Users sometimes create lengthy prompts filled with incongruous elements in an attempt to explain or justify the unexpected images generated by the AI. As a result, they may attribute the outcomes to esoteric reasons or even create myths, seeking explanations within their prompt writing. By focusing on crafting clear and succinct prompts, users can avoid this pitfall and improve their chances of obtaining the desired output.

It is crucial to structure prompts by placing the most significant words at the forefront, as the impact of each word decreases with its position. Keep in mind that the influence of words in the latter part of the prompt wanes, leading to a reduced effect on the final outcome.

Punctuation
-----------

Modifying punctuation in a prompt can significantly influence generative AI models' outputs, as every character, including punctuation, is crucial in determining the generated result. This is due to tokenization - the process of breaking text data into smaller units called tokens, such as words, characters, or subwords.

Altered punctuation changes token interactions, leading to different outputs. The AI model, trained on diverse data, is sensitive to these variations in token or punctuation arrangements. The tokenized input undergoes "self-attention" during processing, where each token, represented by neural network weights, interacts with surrounding tokens across multiple layers. These layers are compressed into a compact latent representation, combined with random image noise using dot product operations, and iteratively refined by the diffusion network to generate coherent outputs based on the model's training data.

Character Set and Case Insensitivity       
-------------------------------------------

It's advisable to phrase your prompt in English, as the majority of the training data set consists of English image-text pairs. 

Keep in mind that prompts are case-insensitive, so using all lowercase letters might be more convenient and consistent.

Classifier-Free Guidance Scale
------------------------------

Classifier-free guidance is a method that generates images without requiring an extra classifier. In traditional methods, a diffusion model's score estimate is mixed with the input gradient of a classifier to improve image quality. However, this requires training an extra classifier, which can be challenging. 

To address this, classifier-free guidance combines conditional and unconditional diffusion model scores to enhance image quality and balance sample diversity. This method enables better control over output images, generating high-quality results that closely match input prompts while maintaining diversity.

Seed
----

The seed is a numeric value that initializes the random number generator used by the AI model during the image generation process.

The default seed value on BoostPixels is set to -1 to ensure that you'll get a unique generated image every time you run the process, without the need to manually change the seed value. However, if you prefer, you can set a specific seed value other than -1. 

By setting a specific seed value, you can reproduce the same generated image multiple times, given the same other input parameters. This is useful when you want to share your results with others, compare different settings, or revisit a particular output.

Limitations and challenges
--------------------------

Text generation is not possible due to the inherent discrete and structured nature of textual data, which consists of individual symbols arranged in a specific order. This characteristic distinguishes it from image generation.

One prevalent challenge is accurately generating hands. The current state of images produced by AI models is often misshapen or appears unnatural. The primary reason for this issue is because hands are composed of numerous bones, joints, and muscles that work together to enable a wide range of complex motions and gestures.

To improve the generation of hands in AI-generated images, models need to be trained on more diverse and representative datasets, capturing the complexity of hands in various poses, orientations, and gestures. This could involve incorporating more detailed annotations or even 3D models.

Controlling the specific characteristics or features of images generated by diffusion models can be challenging. While latent space guidance by text prompts can offer some level of control, achieving fine-grained control over the output (e.g., specific attributes, styles, or content) can be difficult.

Diffusion models are non-deterministic, meaning that they can produce different outputs for the same input due to their inherent randomness. This characteristic can fuel superstition and lead people to attribute successful image outputs to unrelated factors. The phenomenon is reminiscent of Skinner's operant conditioning experiment, where subjects developed superstitious behaviors by associating random events with reinforcement.