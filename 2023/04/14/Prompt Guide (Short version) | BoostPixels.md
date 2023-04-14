> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [boostpixels.com](https://boostpixels.com/short-guide)

> When starting out, people might think typing just one word will generate a cool image. Although AI mo......

When starting out, people might think typing just one word will generate a cool image. Although AI models are getting better and better at understanding what we want, they can't read our minds yet. 

So, if you want to generate an image that looks like what you have in mind, you need to write the right prompt.

Empty Prompt
------------

After training, an empty prompt in BoostPixels results in an image that closely mirrors the photo you uploaded for training.

![](https://boostpixels.com/sites/default/files/inline-images/image_20.png)Prompt "" (empty)

Default word the model is trained on is "**ftpdnx**". This word is automatically included as the start of the prompt, so there's no need to enter it manually every time you write a prompt.

Word Weight
-----------

Weight in words means how much a word affects the image being generated. Important words should come first because they have more influence, and each word after weighs a little less.

If you want a word to matter more, put it in parentheses "()". This helps the AI focus on what you really want in your image.

![](https://boostpixels.com/sites/default/files/inline-images/image_19.png)Prompt: "(((bald)))"

When you use too much weight on a word, the model will only pay attention to that word, and usually, weird images will be generated.

Not all words have the same starting weight. Some words are more powerful because they appear more frequently in the training dataset. You need to balance the importance of different words in your instructions, considering how much impact each word has.

If the generated image doesn't look like you, try adding"(ftpdnx person)" to your prompt. This helps the AI understand that it should use the photo that the model was trained on.

Medium and Style
----------------

Specify the style and medium you want, as these factors greatly impact the final result. If you don't provide this information, the AI will simply use the style of the input image.

Some popular styles include hyperrealistic, surrealistic, cartoon, watercolor, sketch, and concept art. 

Mediums can range from photography and illustration to painting and 3D rendering.

![](https://boostpixels.com/sites/default/files/inline-images/image_13.png)Prompt: "(((bald))) watercolour painting"

Adding Artists
--------------

By including an artist's name, such as"painting by Rembrandt" in your prompt, you can generate images with a specific artistic touch.

![](https://boostpixels.com/sites/default/files/inline-images/image_12.png)Prompt: "((bald)) painting by Rembrandt"

Quality Attributes
------------------

You can define different attributes like resolution, quality, and lighting.

Including words such as 1080 HD, 4K UHD, and 8K UHD to indicate the desired resolution can improve clarity, while terms like "sharp" and "focus" help to enhance details.

Lighting, such as natural light, fluorescent light, LED light, and incandescent light, sets a certain ambiance.

![](https://boostpixels.com/sites/default/files/inline-images/image_15.png)Prompt: "(((bald))) photo Hyperrealistic 4K UHD Sharp Focus LED light"

Context
-------

Creating interesting or funny images with AI often involves placing the subject in a different context. For example, you can generate images with the subject in Venice or Paris as the background.

![](https://boostpixels.com/sites/default/files/inline-images/image_16.png)Prompt: "((portrait)) photo hyperrealistic 4K UHD sharp focus LED light intricate Venice background"

The "as" construction can be used to transform a subject into something else, such as Santa Claus. This can create fun and unique images. By playing around with different concepts, you can create cool and interesting pictures.

![](https://boostpixels.com/sites/default/files/inline-images/image_17.png)Prompt: "as (santa claus) photo hyperrealistic 4K UHD sharp focus LED light intricate background"

AI models don't understand logic like humans do. Therefore, if you give them a prompt like"a person riding a horse beneath an elephant"they won't interpret it the way you expect.

Color
-----

You can write color to generate images containing it. Be careful because it can be hard to change the color of just one part of the image. 

Emotions
--------

Emotional words can help create the mood or atmosphere in AI-generated images. Using different words can make an image feel positive (such as joyful, cheerful) or negative (like terrifying, eerie).

Perspective and Distance
------------------------

Provide clear instructions about how close or far things should be in the generated image. Otherwise, the AI might generate an image that is too similar to the training image.

To generate a full-body image of a person, include words like "jacket," "belt," and "shoes" to ensure the entire figure is captured. You can also use words associated with the type of image you want, such as the names of photographers known for taking photos with a particular distance and perspective.

![](https://boostpixels.com/sites/default/files/inline-images/image_21.png)Prompt: "wearing a brown leather jacket and belt and shoes, forest background, in the style of Brandon Stanton, Humans of New York"

Faces
-----

Generating accurate faces in images with distant people is a tough challenge. One way to improve this is by creating a higher-resolution image and selecting the "Enhance face" option.

The image and face become clearer and more detailed. However, this process takes more time and resources, so it costs more. A high-resolution image (1024 pixels) takes about 8 seconds and 4 credits, while a lower-resolution image (512 pixels) only takes 2 seconds and 1 credit.

The additional effort and cost help to make faces look better and more accurate in the final image.

Negative Prompts
----------------

BoostPixels has a default setting that uses the negative prompt  "cartoon 3d hands deformed ugly" to help generate better images. However, users can choose a different option or create their own.

Negative words do not make things disappear from an image; instead, they reduce the importance of certain words.

Power of Repetition
-------------------

When creating images using artificial intelligence, it is normal not to get the perfect image on the first try. Don't be discouraged if it takes multiple attempts to achieve the desired result, as this is normal due to random noise in the AI model.

Instead of perfecting the prompt, it can be helpful to generate a large number of images using the same prompt to increase the chances of obtaining the desired outcome.

Length of Prompts
-----------------

Prompts are limited to 256 characters. Writing long prompts does not necessarily create better results. It is important to find the right balance between being concise and providing enough information for the AI to generate the desired outcome.

Punctuation
-----------

Altering punctuation in a text prompt influences AI-generated outcomes because each character, even commas or periods, matters. Thus, changing just a comma or period may create a completely different image.

Character Set and Case Insensitivity       
-------------------------------------------

Write your prompts in English because most of the model's training data comes from the English language.

It does not matter whether you write in all lowercase or with capital letters because the model does not differentiate between them.

Classifier-Free Guidance Scale
------------------------------

When you increase the Classifier-free guidance scale, the generated images look more like the input prompt, but the variety of images produced may be less.

Seed
----

The seed is a numeric value used as a starting point for the model to create random noise. By default, the seed value is set to -1. This means that every time you generate an image, you'll get a completely new image, even if you use the same prompt and settings. But if you want to reproduce the same image, you can set a specific seed value.

Limitations
-----------

Text creation is not possible because it consists of individual symbols that follow a specific order and structure. This discrete and structured nature of text is quite different from the nature of images.

Hands are often misshapen or appear unnatural because they are complex with many details. It is difficult to capture the shape, size, and position of fingers and joints properly.

AI can produce varying results even with the same prompt. This can lead people to believe that certain factors matter when they do not, kind of like believing that wearing a lucky shirt helps your team win a game.