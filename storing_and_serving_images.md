
Source: http://highscalability.com/blog/2016/4/20/how-twitter-handles-3000-images-per-second.html

Gist:
1. Decouple media upload from tweets
2. Move handles not blobs: Don't move the media between internal services. It wastes the internal bandwidth.
Instead, store the data and refer it with a handle
3. Segemented resumable uploads: upload sessions should return a mediaId which the client can use to resume an interrupted
upload.
5. Image pipelines: Once the image is uploaded, the post processing tasks like face detection, content violation detection,
variant generation can be run asynchronously.
4. On demand variant generation: Keep the original image until deletion. Variants can be deleted after few days and generated
on demand when requested. For twitter like traffic, most of the requests are for images within last 15 days.
5. Progressive Jpegs - Clients should support progressive JPEG format. It is slower to encode but for for read heavy
traffic it is useful as encoding needs to be done only once. https://github.com/facebook/fresco