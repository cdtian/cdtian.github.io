---
layout: post
title:  "some way download file use java"
date:   2019-08-14 17:07:54
categories: java
tags: io nio
excerpt: 
mathjax: true
---


this is some way to download file java example
include io,nio, or use third party jar such as common-io,async-http-client

```java
/**
     * custom download file use java io
     *
     * @param url
     * @param localFilename
     */
    public static void downloadWithJavaIO(String url, String localFilename) {

        try (BufferedInputStream in = new BufferedInputStream(new URL(url).openStream());
             FileOutputStream fileOutputStream = new FileOutputStream(localFilename)) {
            byte dataBuffer[] = new byte[1024];
            int bytesRead;
            while ((bytesRead = in.read(dataBuffer, 0, 1024)) != -1) {
                fileOutputStream.write(dataBuffer, 0, bytesRead);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * download file use java 7 io
     *
     * @param url
     * @param localFilename
     */
    public static void downloadWithJava7IO(String url, String localFilename) {
        try (InputStream in = new URL(url).openStream()) {
            Files.copy(in, Paths.get(localFilename), StandardCopyOption.REPLACE_EXISTING);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * download file use java nio
     *
     * @param fileURL
     * @param localFilename
     * @throws IOException
     */
    public static void downloadWithJavaNIO(String fileURL, String localFilename) throws IOException {
        URL url = new URL(fileURL);
        try (ReadableByteChannel readableByteChannel = Channels.newChannel(url.openStream());
             FileOutputStream fileOutputStream = new FileOutputStream(localFilename);
             FileChannel fileChannel = fileOutputStream.getChannel()) {
            fileChannel.transferFrom(readableByteChannel, 0, Long.MAX_VALUE);
        }
    }

    /**
     * you need commons-io jar
     *
     * @param url
     * @param localFilename
     */
    public static void downloadWithApacheCommons(String url, String localFilename) {

        int CONNECT_TIMEOUT = 10000;
        int READ_TIMEOUT = 10000;
        try {
            FileUtils.copyURLToFile(new URL(url), new File(localFilename), CONNECT_TIMEOUT, READ_TIMEOUT);
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

    /**
     * you need import async-http-client jar
     *
     * @param url
     * @param localFilename
     * @throws ExecutionException
     * @throws InterruptedException
     * @throws IOException
     */
    public static void downloadWithAHC(String url, String localFilename) throws ExecutionException, InterruptedException, IOException {

        FileOutputStream stream = new FileOutputStream(localFilename);
        AsyncHttpClient client = Dsl.asyncHttpClient();

        client.prepareGet(url)
                .execute(new AsyncCompletionHandler<FileOutputStream>() {

                    @Override
                    public State onBodyPartReceived(HttpResponseBodyPart bodyPart) throws Exception {
                        stream.getChannel()
                                .write(bodyPart.getBodyByteBuffer());
                        return State.CONTINUE;
                    }

                    @Override
                    public FileOutputStream onCompleted(Response response) throws Exception {
                        return stream;
                    }
                })
                .get();

        stream.getChannel().close();
        client.close();
    }
```



