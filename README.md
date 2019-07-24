# rtsp_ijk

# **Скомпилированная версия IJK Player 0.8.8 для RTSP, содержит все основные кодеки.**


## **Компиляция для Linux:**

sudo apt-get install yasm

export ANDROID_SDK=<путь до SDK>

export ANDROID_NDK=<путь до [NDK 10](https://dl.google.com/android/repository/android-ndk-r10e-windows-x86_64.zip?hl=ru "NDK 10"))>



    git clone https://github.com/Bilibili/ijkplayer.git ijkplayer-android
    cd ijkplayer-android
    ./init-android.sh

    export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --enable-protocol=rtp"
    export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --enable-protocol=tcp"
    export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --enable-demuxer=rtsp"
    export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --enable-demuxer=sdp"
    export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --enable-demuxer=rtp"

    cd config
Поместить файл [rtsp-lite.sh](https://github.com/X1opya/rtsp_ijk/blob/master/rtsp-lite.sh) в папку config



    rm module.sh
    ln -s rtsp-lite.sh module.sh
    cd ..

Заменить в файле ijkmedia/ijkplayer/ff_ffplay.c метод packet_queue_get_or_buffering

```c
static int packet_queue_get_or_buffering(FFPlayer *ffp, PacketQueue *q, AVPacket *pkt, int *serial, int *finished)
{
    assert(finished);
    if (!ffp->packet_buffering)
        return packet_queue_get(q, pkt, 1, serial);

    while (1) {
        int new_packet = packet_queue_get(q, pkt, 0, serial);
        if (new_packet < 0)
            return -1;
        else if (new_packet == 0) {
            if (q->is_buffer_indicator && !*finished)
                ffp_toggle_buffering(ffp, 1);
            new_packet = packet_queue_get(q, pkt, 1, serial);
            if (new_packet < 0)
                return -1;
        }

        if (*finished == *serial) {
            av_packet_unref(pkt);
            continue;
        }
        else
            break;
    }

    return 1;
}
```

на

```c
static int packet_queue_get_or_buffering(FFPlayer *ffp, PacketQueue *q, AVPacket *pkt, int *serial, int *finished)
{
    while (1) {
    int new_packet = packet_queue_get(q, pkt, 1, serial);
    if (new_packet < 0)
    {
        new_packet = packet_queue_get(q, pkt, 0, serial);
        if(new_packet < 0)
        return -1;
    }
    else if (new_packet == 0) {
        if (!finished)
            ffp_toggle_buffering(ffp, 1);
        new_packet = packet_queue_get(q, pkt, 1, serial);
        if (new_packet < 0)
            return -1;
    }
    if (*finished == *serial) {
        av_free_packet(pkt);
        continue;
    }
    else
        break;
    }
    return 1;
}
```



    cd android/contrib
    ./compile-ffmpeg.sh clean
    ./compile-ffmpeg.sh all
    
    cd ..
    ./compile-ijk.sh all
    
    
    
## **Импорт в проект:**

**A) Импортировать как модули android/ijkplayer**

**Б) Сгенерировать arr файлы от каждого модуля**
