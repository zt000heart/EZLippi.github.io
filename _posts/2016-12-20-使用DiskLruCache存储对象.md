---
layout: post
title: "使用DiskLruCache实现硬盘对象存储"
description: "硬盘存储"
category: 记一发代码
tags: [width，height]
---

    public class DiskLruManager {

        public static int size = 1024 * 1024 * 10;  //10M
        public static int DEFAULT_VERSION = 1;

        public static DiskLruCache getDiskLruCache(Context context, String name, int version){
            try {
                File file = getDiskCacheDir(context,name);
                if (!file.exists()) {
                    file.mkdirs();
                }
                DiskLruCache diskLruCache = DiskLruCache.open(file, version, 1, size);
                return diskLruCache;
            } catch (IOException e) {
                e.printStackTrace();
            }
            return null;
        }

        public static void writeObject(Serializable object, Context context , String name, String key) {
            try {
                DiskLruCache diskLruCache = getDiskLruCache(context ,name ,DEFAULT_VERSION);
                DiskLruCache.Editor editor = diskLruCache.edit(key);
                OutputStream outputStream = editor.newOutputStream(0);
                if (writeObjectToStream(object, outputStream)) {
                    editor.commit();
                } else {
                    editor.abort();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

        public static <T> T readObject(Context context ,Class<T> clazz, String name, String key){
            try {
                DiskLruCache diskLruCache = getDiskLruCache(context ,name ,DEFAULT_VERSION);
                DiskLruCache.Snapshot snapshot =  diskLruCache.get(key);
                if(snapshot == null){
                    return null;
                }
                InputStream inputStream = snapshot.getInputStream(0);
                ObjectInputStream objectInputStream = new ObjectInputStream(inputStream);
                Object object = objectInputStream.readObject();
                return (T) object;
            } catch (IOException | ClassNotFoundException | ClassCastException e) {
                e.printStackTrace();
                return null;
            }
        }

        private static boolean writeObjectToStream(Serializable obj,OutputStream outputStream){
            try {
                ObjectOutputStream objectOutputStream = new ObjectOutputStream(outputStream);
                objectOutputStream.writeObject(obj);
                objectOutputStream.close();
                return true;
            } catch (IOException e) {
                e.printStackTrace();
                return false;
            }
        }

        private static File getDiskCacheDir(Context context, String uniqueName) {
            String cachePath;
            //如果sd卡存在并且没有被移除
            if (Environment.MEDIA_MOUNTED.equals(Environment
                    .getExternalStorageState())
                    || !Environment.isExternalStorageRemovable()) {
                cachePath = context.getExternalCacheDir().getPath();
            } else {
                cachePath = context.getCacheDir().getPath();
            }
            return new File(cachePath + File.separator + uniqueName);
        }
    }
    
通过name和key的形式存储，使用Serializable对象序列化，也可以使用Json。。提升DEFAULT_VERSION将会废弃以前的存储内容。