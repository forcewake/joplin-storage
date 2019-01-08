Machine Learning is Fun! Part 4: Modern Face Recognition with Deep Learning

**_Update:_** _This article is part of a series. Check out the full series:_ [_Part 1_](https://medium.com/@ageitgey/machine-learning-is-fun-80ea3ec3c471)_,_ [_Part 2_](https://medium.com/@ageitgey/machine-learning-is-fun-part-2-a26a10b68df3)_,_ [_Part 3_](https://medium.com/@ageitgey/machine-learning-is-fun-part-3-deep-learning-and-convolutional-neural-networks-f40359318721)_,_ [_Part 4_](https://medium.com/@ageitgey/machine-learning-is-fun-part-4-modern-face-recognition-with-deep-learning-c3cffc121d78)_,_ [_Part 5_](https://medium.com/@ageitgey/machine-learning-is-fun-part-5-language-translation-with-deep-learning-and-the-magic-of-sequences-2ace0acca0aa)_,_ [_Part 6_](https://medium.com/@ageitgey/machine-learning-is-fun-part-6-how-to-do-speech-recognition-with-deep-learning-28293c162f7a)_,_ [_Part 7_](https://medium.com/@ageitgey/abusing-generative-adversarial-networks-to-make-8-bit-pixel-art-e45d9b96cee7) _and_ [_Part 8_](https://medium.com/@ageitgey/machine-learning-is-fun-part-8-how-to-intentionally-trick-neural-networks-b55da32b7196)_! You can also read this article in_ [_普通话_](https://zhuanlan.zhihu.com/p/24567586)_,_ [_Русский_](http://algotravelling.com/ru/%D0%BC%D0%B0%D1%88%D0%B8%D0%BD%D0%BD%D0%BE%D0%B5-%D0%BE%D0%B1%D1%83%D1%87%D0%B5%D0%BD%D0%B8%D0%B5-%D1%8D%D1%82%D0%BE-%D0%B2%D0%B5%D1%81%D0%B5%D0%BB%D0%BE-4/)_,_ [_한국어_](https://medium.com/@jongdae.lim/기계-학습-machine-learning-은-즐겁다-part-4-63ed781eee3c)_,_ [_Português_](https://medium.com/machina-sapiens/aprendizagem-de-m%C3%A1quina-%C3%A9-divertido-parte-4-reconhecimento-facial-moderno-com-deep-learning-72525d9684c2)_,_ [_Tiếng Việt_](https://viblo.asia/p/machine-learning-that-thu-vi-4-tu-dong-tag-ten-ban-be-ORNZqPDqK0n) _or_ [_Italiano_](https://medium.com/botsupply/il-machine-learning-è-divertente-parte-4-c707feee1cf8)_._

**_Giant update:_**  [_I’ve written a new book based on these articles_](https://www.machinelearningisfun.com/get-the-book/)_! It not only expands and updates all my articles, but it has tons of brand new content and lots of hands-on coding projects._ [_Check it out now_](https://www.machinelearningisfun.com/get-the-book/)_!_

Have you noticed that Facebook has developed an uncanny ability to recognize your friends in your photographs? In the old days, Facebook used to make you to tag your friends in photos by clicking on them and typing in their name. Now as soon as you upload a photo, Facebook tags everyone for you _like magic_:

![](../../_resources/67e4a42afcac44d39672da5a9669a536.jpg)

![](../../_resources/d0b8c34e8a56443b8542480cc06bad09.gif)

![](../../_resources/d0b8c34e8a56443b8542480cc06bad09.gif)

Facebook automatically tags people in your photos that you have tagged before. I’m not sure if this is helpful or creepy!

This technology is called face recognition. Facebook’s algorithms are able to recognize your friends’ faces after they have been tagged only a few times. It’s pretty amazing technology — Facebook can recognize faces with [98% accuracy](https://research.facebook.com/publications/deepface-closing-the-gap-to-human-level-performance-in-face-verification/) which is pretty much as good as humans can do!

Let’s learn how modern face recognition works! But just recognizing your friends would be too easy. We can push this tech to the limit to solve a more challenging problem — telling [Will Ferrell](https://en.wikipedia.org/wiki/Will_Ferrell) (famous actor) apart from [Chad Smith](https://en.wikipedia.org/wiki/Chad_Smith) (famous rock musician)!

![](../../_resources/0f7e86ec35674883a8de582e4ac573b2.jpg)

![](../../_resources/6d6a8e232d0c451989c0e6d4f2f61b9a.jpeg)

One of these people is Will Farrell. The other is Chad Smith. I swear they are different people!

* * *

### How to use Machine Learning on a Very Complicated Problem

So far in [Part 1](https://medium.com/@ageitgey/machine-learning-is-fun-80ea3ec3c471), [2](https://medium.com/@ageitgey/machine-learning-is-fun-part-2-a26a10b68df3) and [3](https://medium.com/@ageitgey/machine-learning-is-fun-part-3-deep-learning-and-convolutional-neural-networks-f40359318721), we’ve used machine learning to solve isolated problems that have only one step — [estimating the price of a house](https://medium.com/@ageitgey/machine-learning-is-fun-80ea3ec3c471), [generating new data based on existing data](https://medium.com/@ageitgey/machine-learning-is-fun-part-2-a26a10b68df3) and [telling if an image contains a certain object](https://medium.com/@ageitgey/machine-learning-is-fun-part-3-deep-learning-and-convolutional-neural-networks-f40359318721). All of those problems can be solved by choosing one machine learning algorithm, feeding in data, and getting the result.

But face recognition is really a series of several related problems:

1.  First, look at a picture and find all the faces in it
2.  Second, focus on each face and be able to understand that even if a face is turned in a weird direction or in bad lighting, it is still the same person.
3.  Third, be able to pick out unique features of the face that you can use to tell it apart from other people— like how big the eyes are, how long the face is, etc.
4.  Finally, compare the unique features of that face to all the people you already know to determine the person’s name.

As a human, your brain is wired to do all of this automatically and instantly. In fact, humans are _too good_ at recognizing faces and end up seeing faces in everyday objects:

![](../../_resources/7820b2a0c95545bf8053c2dcedb23470.jpg)

![](../../_resources/34cabca4659245419f3937159bd8610a.jpeg)

Computers are not capable of this kind of high-level generalization (_at least not yet…_), so we have to teach them how to do each step in this process separately.

We need to build a _pipeline_ where we solve each step of face recognition separately and pass the result of the current step to the next step. In other words, we will chain together several machine learning algorithms:

![](../../_resources/836ebe4192324f20b3b8b4ce0e70a12d.jpg)

![](../../_resources/ee04f7116d434cb7a630fa347d3714a9.gif)

How a basic pipeline for detecting faces might work

* * *

### Face Recognition — Step by Step

Let’s tackle this problem one step at a time. For each step, we’ll learn about a different machine learning algorithm. I’m not going to explain every single algorithm completely to keep this from turning into a book, but you’ll learn the main ideas behind each one and you’ll learn how you can build your own facial recognition system in Python using [OpenFace](https://cmusatyalab.github.io/openface/) and [dlib](http://dlib.net/).

#### Step 1: Finding all the Faces

The first step in our pipeline is _face detection_. Obviously we need to locate the faces in a photograph before we can try to tell them apart!

If you’ve used any camera in the last 10 years, you’ve probably seen face detection in action:

![](../../_resources/de930a66f9434ea3bced3052a40bff2b.png)

![](../../_resources/efc73abba18b442da92c70c15ac1fe72.png)

Face detection is a great feature for cameras. When the camera can automatically pick out faces, it can make sure that all the faces are in focus before it takes the picture. But we’ll use it for a different purpose — finding the areas of the image we want to pass on to the next step in our pipeline.

Face detection went mainstream in the early 2000's when Paul Viola and Michael Jones invented a [way to detect faces](https://en.wikipedia.org/wiki/Viola%E2%80%93Jones_object_detection_framework) that was fast enough to run on cheap cameras. However, much more reliable solutions exist now. We’re going to use [a method invented in 2005](http://lear.inrialpes.fr/people/triggs/pubs/Dalal-cvpr05.pdf) called Histogram of Oriented Gradients — or just **_HOG_** for short.

To find faces in an image, we’ll start by making our image black and white because we don’t need color data to find faces:

![](../../_resources/66b07af3d5154a2c820afaa6994901c4.jpg)

![](../../_resources/381a055f919f48389fc70d357d86d54a.jpeg)

Then we’ll look at every single pixel in our image one at a time. For every single pixel, we want to look at the pixels that directly surrounding it:

![](../../_resources/61837c296cac4144ac4ad8a8f473f291.jpg)

![](../../_resources/53bd1746f64940b8b5d313a5660bf3c5.gif)

Our goal is to figure out how dark the current pixel is compared to the pixels directly surrounding it. Then we want to draw an arrow showing in which direction the image is getting darker:

![](../../_resources/866ba98a5505403f9446cac4aa67e08f.jpg)

![](../../_resources/79a5f865e353493d970c7ecd68aea655.gif)

Looking at just this one pixel and the pixels touching it, the image is getting darker towards the upper right.

If you repeat that process for **every single pixel** in the image, you end up with every pixel being replaced by an arrow. These arrows are called _gradients_ and they show the flow from light to dark across the entire image:

![](../../_resources/2dd73251dc7548b4961e7b860e89586a.jpg)

![](../../_resources/d90ce453f55e4d96ab4f9be3a796403f.gif)

This might seem like a random thing to do, but there’s a really good reason for replacing the pixels with gradients. If we analyze pixels directly, really dark images and really light images of the same person will have totally different pixel values. But by only considering the _direction_ that brightness changes, both really dark images and really bright images will end up with the same exact representation. That makes the problem a lot easier to solve!

But saving the gradient for every single pixel gives us way too much detail. We end up [missing the forest for the trees](https://en.wiktionary.org/wiki/see_the_forest_for_the_trees). It would be better if we could just see the basic flow of lightness/darkness at a higher level so we could see the basic pattern of the image.

To do this, we’ll break up the image into small squares of 16x16 pixels each. In each square, we’ll count up how many gradients point in each major direction (how many point up, point up-right, point right, etc…). Then we’ll replace that square in the image with the arrow directions that were the strongest.

The end result is we turn the original image into a very simple representation that captures the basic structure of a face in a simple way:

![](../../_resources/8e4004d2dc6d4ea1a02325b0addd3490.jpg)

![](../../_resources/c80faf27b500401fba540fa43a5c81f7.gif)

The original image is turned into a HOG representation that captures the major features of the image regardless of image brightnesss.

To find faces in this HOG image, all we have to do is find the part of our image that looks the most similar to a known HOG pattern that was extracted from a bunch of other training faces:

![](../../_resources/a47d294205544fbeac46220759cffadf.png)

![](../../_resources/8ec52f564baf49bcb55c3db1b2208f6e.png)

Using this technique, we can now easily find faces in any image:

![](../../_resources/f27c66ed799e411b8a506d3bb3505802.jpg)

![](../../_resources/f419c99a4a3849009ee15c0677708d96.jpeg)

If you want to try this step out yourself using Python and dlib, [here’s code](https://gist.github.com/ageitgey/1c1cb1c60ace321868f7410d48c228e1) showing how to generate and view HOG representations of images.

#### Step 2: Posing and Projecting Faces

Whew, we isolated the faces in our image. But now we have to deal with the problem that faces turned different directions look totally different to a computer:

![](../../_resources/b9c9c56b7d284ed2abc3f60d25edfbdd.png)

![](../../_resources/0a3eb01e7a374ae9a2c38377f031fe8d.png)

Humans can easily recognize that both images are of Will Ferrell, but computers would see these pictures as two completely different people.

To account for this, we will try to warp each picture so that the eyes and lips are always in the sample place in the image. This will make it a lot easier for us to compare faces in the next steps.

To do this, we are going to use an algorithm called **face landmark estimation**. There are lots of ways to do this, but we are going to use the approach [invented in 2014 by Vahid Kazemi and Josephine Sullivan.](http://www.csc.kth.se/~vahidk/papers/KazemiCVPR14.pdf)

The basic idea is we will come up with 68 specific points (called _landmarks_) that exist on every face — the top of the chin, the outside edge of each eye, the inner edge of each eyebrow, etc. Then we will train a machine learning algorithm to be able to find these 68 specific points on any face:

![](../../_resources/68b61bf5824940bea8f4810c5fc3229f.png)

![](../../_resources/263da344cfb64bbc945374684f33366f.png)

The 68 landmarks we will locate on every face. This image was created by [Brandon Amos](http://bamos.github.io/) of CMU who works on [OpenFace](https://github.com/cmusatyalab/openface).

Here’s the result of locating the 68 face landmarks on our test image:

![](../../_resources/1be5a228980446cf91a679e0f7b00c9c.jpg)

![](../../_resources/0bf9e79a12184ea49efde711cebd7921.jpeg)

**PROTIP**: You can also use this same technique to implement your own version of Snapchat’s real-time 3d face filters!

Now that we know were the eyes and mouth are, we’ll simply rotate, scale and [shear](https://en.wikipedia.org/wiki/Shear_mapping#/media/File:VerticalShear_m%3D1.25.svg) the image so that the eyes and mouth are centered as best as possible. We won’t do any fancy 3d warps because that would introduce distortions into the image. We are only going to use basic image transformations like rotation and scale that preserve parallel lines (called [affine transformations](https://en.wikipedia.org/wiki/Affine_transformation)):

![](../../_resources/6fdabd1bfa9841ca899e775bd7848bf1.png)

![](../../_resources/bf859385d055448db80c5ce1d5bf138c.png)

Now no matter how the face is turned, we are able to center the eyes and mouth are in roughly the same position in the image. This will make our next step a lot more accurate.

If you want to try this step out yourself using Python and dlib, here’s the [code for finding face landmarks](https://gist.github.com/ageitgey/ae340db3e493530d5e1f9c15292e5c74) and here’s the [code for transforming the image](https://gist.github.com/ageitgey/82d0ea0fdb56dc93cb9b716e7ceb364b) using those landmarks.

#### Step 3: Encoding Faces

Now we are to the meat of the problem — actually telling faces apart. This is where things get really interesting!

The simplest approach to face recognition is to directly compare the unknown face we found in Step 2 with all the pictures we have of people that have already been tagged. When we find a previously tagged face that looks very similar to our unknown face, it must be the same person. Seems like a pretty good idea, right?

There’s actually a huge problem with that approach. A site like Facebook with billions of users and a trillion photos can’t possibly loop through every previous-tagged face to compare it to every newly uploaded picture. That would take way too long. They need to be able to recognize faces in milliseconds, not hours.

What we need is a way to extract a few basic measurements from each face. Then we could measure our unknown face the same way and find the known face with the closest measurements. For example, we might measure the size of each ear, the spacing between the eyes, the length of the nose, etc. If you’ve ever watched a bad crime show like [CSI](https://en.wikipedia.org/wiki/CSI:_Crime_Scene_Investigation), you know what I am talking about:

![](../../_resources/46dda842c2464471a010ff8f9a9f1f15.jpg)

![](https://cdn-images-1.medium.com/max/1600/1*_GNyjR3JlPoS9grtIVmKFQ.gif)

Just like TV! So real! #science

#### The most reliable way to measure a face

Ok, so which measurements should we collect from each face to build our known face database? Ear size? Nose length? Eye color? Something else?

It turns out that the measurements that seem obvious to us humans (like eye color) don’t really make sense to a computer looking at individual pixels in an image. Researchers have discovered that the most accurate approach is to let the computer figure out the measurements to collect itself. Deep learning does a better job than humans at figuring out which parts of a face are important to measure.

The solution is to train a Deep Convolutional Neural Network ([just like we did in Part 3](https://medium.com/@ageitgey/machine-learning-is-fun-part-3-deep-learning-and-convolutional-neural-networks-f40359318721)). But instead of training the network to recognize pictures objects like we did last time, we are going to train it to generate 128 measurements for each face.

The training process works by looking at 3 face images at a time:

1.  Load a training face image of a known person
2.  Load another picture of the same known person
3.  Load a picture of a totally different person

Then the algorithm looks at the measurements it is currently generating for each of those three images. It then tweaks the neural network slightly so that it makes sure the measurements it generates for #1 and #2 are slightly closer while making sure the measurements for #2 and #3 are slightly further apart:

![](../../_resources/eef15e615fcf46288588520ecce25870.png)

![](../../_resources/1556188e86124cbcb414939031171909.png)

After repeating this step millions of times for millions of images of thousands of different people, the neural network learns to reliably generate 128 measurements for each person. Any ten different pictures of the same person should give roughly the same measurements.

Machine learning people call the 128 measurements of each face an **embedding**. The idea of reducing complicated raw data like a picture into a list of computer-generated numbers comes up a lot in machine learning (especially in language translation). The exact approach for faces we are using [was invented in 2015 by researchers at Google](http://www.cv-foundation.org/openaccess/content_cvpr_2015/app/1A_089.pdf) but many similar approaches exist.

#### Encoding our face image

This process of training a convolutional neural network to output face embeddings requires a lot of data and computer power. Even with an expensive [NVidia Telsa video card](http://www.nvidia.com/object/tesla-supercomputing-solutions.html), it takes [about 24 hours](https://twitter.com/brandondamos/status/757959518433243136) of continuous training to get good accuracy.

But once the network has been trained, it can generate measurements for any face, even ones it has never seen before! So this step only needs to be done once. Lucky for us, the fine folks at [OpenFace](https://cmusatyalab.github.io/openface/) already did this and they [published several trained networks](https://github.com/cmusatyalab/openface/tree/master/models/openface) which we can directly use. Thanks [Brandon Amos](http://bamos.github.io/) and team!

So all we need to do ourselves is run our face images through their pre-trained network to get the 128 measurements for each face. Here’s the measurements for our test image:

![](../../_resources/b3e87851b1da4db89a5eb7f5437a7226.png)

![](../../_resources/a788e7f035754d7f9e285d82ee2e7536.png)

So what parts of the face are these 128 numbers measuring exactly? It turns out that we have no idea. It doesn’t really matter to us. All that we care is that the network generates nearly the same numbers when looking at two different pictures of the same person.

If you want to try this step yourself, OpenFace [provides a lua script](https://github.com/cmusatyalab/openface/blob/master/batch-represent/batch-represent.lua) that will generate embeddings all images in a folder and write them to a csv file. You [run it like this](https://gist.github.com/ageitgey/ddbae3b209b6344a458fa41a3cf75719).

#### Step 4: Finding the person’s name from the encoding

This last step is actually the easiest step in the whole process. All we have to do is find the person in our database of known people who has the closest measurements to our test image.

You can do that by using any basic machine learning classification algorithm. No fancy deep learning tricks are needed. We’ll use a simple linear [SVM classifier](https://en.wikipedia.org/wiki/Support_vector_machine), but lots of classification algorithms could work.

All we need to do is train a classifier that can take in the measurements from a new test image and tells which known person is the closest match. Running this classifier takes milliseconds. The result of the classifier is the name of the person!

So let’s try out our system. First, I trained a classifier with the embeddings of about 20 pictures each of Will Ferrell, Chad Smith and Jimmy Falon:

![](../../_resources/a4daceac572344d19b55a9107d33d939.jpg)

![](../../_resources/eae5dc94db7143c598c2719b0e20c95f.jpeg)

Sweet, sweet training data!

Then I ran the classifier on every frame of the famous youtube video of [Will Ferrell and Chad Smith pretending to be each other](https://www.youtube.com/watch?v=EsWHyBOk2iQ) on the Jimmy Fallon show:

![](../../_resources/f329fd7aa2a04c5f913212bdfd5a75d0.jpg)

![](https://cdn-images-1.medium.com/max/1600/1*woPojJbd6lT7CFZ9lHRVDw.gif)

It works! And look how well it works for faces in different poses — even sideways faces!

### Running this Yourself

Let’s review the steps we followed:

1.  Encode a picture using the HOG algorithm to create a simplified version of the image. Using this simplified image, find the part of the image that most looks like a generic HOG encoding of a face.
2.  Figure out the pose of the face by finding the main landmarks in the face. Once we find those landmarks, use them to warp the image so that the eyes and mouth are centered.
3.  Pass the centered face image through a neural network that knows how to measure features of the face. Save those 128 measurements.
4.  Looking at all the faces we’ve measured in the past, see which person has the closest measurements to our face’s measurements. That’s our match!

Now that you know how this all works, here’s instructions from start-to-finish of how run this entire face recognition pipeline on your own computer:

**_UPDATE 4/9/2017:_** You can still follow the steps below to use OpenFace. However, I’ve released a new Python-based face recognition library called [face_recognition](https://github.com/ageitgey/face_recognition#face-recognition) that is much easier to install and use. So I’d recommend trying out [face_recognition](https://github.com/ageitgey/face_recognition#face-recognition) first instead of continuing below!

I even put together [a pre-configured virtual machine with face_recognition, OpenCV, TensorFlow and lots of other deep learning tools pre-installed](https://medium.com/@ageitgey/try-deep-learning-in-python-now-with-a-fully-pre-configured-vm-1d97d4c3e9b). You can download and run it on your computer very easily. Give the virtual machine a shot if you don’t want to install all these libraries yourself!

_Original OpenFace instructions:_

* * *

If you liked this article, please consider signing up for my Machine Learning is Fun! newsletter:

![](../../_resources/ffb99ea8fa02409fbf2eee563d16f693.jpgkeya19fcc184b9711e1b4764040d3dc5c07width40)

You can also follow me on Twitter at [@ageitgey](https://twitter.com/ageitgey), [email me directly](mailto:ageitgey@gmail.com) or [find me on linkedin](https://www.linkedin.com/in/ageitgey). I’d love to hear from you if I can help you or your team with machine learning.

_Now continue on to_ [_Machine Learning is Fun Part 5_](https://medium.com/@ageitgey/machine-learning-is-fun-part-5-language-translation-with-deep-learning-and-the-magic-of-sequences-2ace0acca0aa)_!_

*   [Machine Learning](https://medium.com/tag/machine-learning?source=post)
*   [Artificial Intelligence](https://medium.com/tag/artificial-intelligence?source=post)
*   [Deep Learning](https://medium.com/tag/deep-learning?source=post)