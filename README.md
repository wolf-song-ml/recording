# 多个视频文件合成画中画效果（Python版）
关键词：ffmpeg 画中画 python

# Step 1 从视频中分离出音频（MP4->mp3）
    def separateMp4ToMp3(tmp):
       mp4 = tmp.replace('.tmp', '.mp4')
       print('---> Separate the video clip {0}'.format(mp4))

       mp3 = tmp.replace('.tmp', '.mp3')
       if os.path.exists(mp3):
          print '\n\t{0} is detected. Skip. \n\tPlease delete .mp3 file if you need re-separate.'.format(mp3)
          return

       cmd = 'ffmpeg -i {0} -f mp3 -vn -loglevel fatal {1}'.format(mp4, mp3)
       print '\t{0}'.format(cmd)

       x = subprocess.Popen(cmd, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)

       for log in x.stdout.readlines():
          print '[ffmpeg info] {0}'.format(log)
       for log in x.stderr.readlines():
          print '[ffmpeg error] {0}'.format(log)

       print '\tSuccess! {0} -> {1}\n'.format(mp4, mp3)

# Step 2 根据时间轴多个音频合成一份音频（MP3->mp3）
    def composeMp3ToMp3(arr = []):
       if len(arr) <=0 :
          print('--->Operate audio array is empty!')
          return

       thisDir = os.path.dirname(arr[0])
       if (os.path.exists(thisDir + "/composeAudio.mp3")):
          print('--->{0}/composeAudio.mp3 is exist, if you need re-gennerate,Please delete it!'.format(thisDir))
          return

       print('---> Compose the audio :')
       var = ''
       for tem in arr:
          if os.path.exists(tem) == False:
             print '\n\t{0} is not exist! \n\tPlease make sure audio file be exist if you need compose.'.format(tem)
             return
          var = var + " -i " + tem

       if var == '':
          print '\n\t{0} is empty. \n\tPlease check .mp3 file if you need compose.'.format(var)
          return

       cmd = 'ffmpeg {0} -filter_complex amix=inputs=2:duration=first:dropout_transition=2 -f mp3 -loglevel fatal {1}/composeAudio.mp3'.format(var, thisDir)
       print '\t{0}'.format(cmd)
       x = subprocess.Popen(cmd, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)

       for log in x.stdout.readlines():
          print '[ffmpeg info] {0}'.format(log)
       for log in x.stderr.readlines():
          print '[ffmpeg error] {0}'.format(log)

       print '\tSuccess! {0} -> {1}\n'.format(var, thisDir + "/composeAudio.mp3")

# Step 3 多个视频合成画中画效果<无声>（MP4->mp4）
        #deal with video:
        def composeMp4ToMp4(arr = []):
           if len(arr) <= 0:
              print('--->Operate video array is empty!')
              return

           thisDir = os.path.dirname(arr[0])
           if (os.path.exists(thisDir + "/composeVideo.mp4")):
              print('--->{0}/composeVideo.mp4 is exist, if you need re-gennerate,Please delete it!'.format(thisDir))
              return

           print('---> Compose the video :')
           var = ''
           temparr = []
           for tem in arr:
              if os.path.exists(tem) == False:
                 print '\n\t{0} is not exist! \n\tPlease make sure video file be exist if you need compose.'.format(tem)
                 return

              #split image
              png = tem.replace('.mp4', '.png')
              tempcmd="ffmpeg -i {0} -ss 00:00:2.435 -loglevel fatal -vframes 1 {1}".format(tem, png)
              print '\t{0}'.format(tempcmd)
              x = subprocess.Popen(tempcmd, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
              x.wait()
              for log in x.stdout.readlines():
                 print'[ffmpeg info] {0}'.format(log)
              for log in x.stderr.readlines():
                 print'[ffmpeg error] {0}'.format(log)

              img = Image.open(png)
              imgSize = img.size
              #ipad
              if (imgSize[0] > imgSize[1]) :
                 temparr.append(tem)
              #mobile
              else:
                 var = var + " -i " + tem
              img.close()

           if (len(temparr) > 0):
              for tem in temparr:
                 var = var + " -i " + tem

           if var == '':
              print '\n\t{0} is empty. \n\tPlease check video file if you need compose.'.format(var)
              return

           cmd = 'ffmpeg ' + var + ' -filter_complex "[1:v]scale=w=176:h=144:force_original_aspect_ratio=decrease[ckout];[0:v]' \
                '[ckout]overlay=x=W-w-10:y=10[out]" -map "[out]" -movflags faststart -loglevel fatal ' + thisDir + '/composeVideo.mp4'.format(var, thisDir)
           print '\t{0}'.format(cmd)
           x = subprocess.Popen(cmd, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)

           for log in x.stdout.readlines():
              print '[ffmpeg info] {0}'.format(log)
           for log in x.stderr.readlines():
              print '[ffmpeg error] {0}'.format(log)

           print '\tSuccess!\n {0} -> {1}\n'.format(var, thisDir + "/composeVideo.mp4")


# Step 4 音频与视频合成
    def communicateAudioVideo(folder):
       if (os.path.exists(folder + "/communicateVideo.mp4")):
          print('--->{0}/communicateVideo.mp4 is exist, if you need re-gennerate,Please delete it!'.format(folder))
          return

       if ((os.path.exists(folder + "/composeVideo.mp4") == False) or
             (os.path.exists(folder + "/composeAudio.mp3") == False)):
          print('--->{0}/composeVideo.mp4  or composeAudio.mp3 must be exist!'.format(folder))
          return

       print('---> Communicate the video :')
       cmd = 'ffmpeg -i ' + folder + '/composeVideo.mp4 -i ' + folder + '/composeAudio.mp3 -f mp4 ' \
             ' -loglevel fatal ' + folder +'/communicateVideo.mp4'
       print '\t{0}'.format(cmd)
       x = subprocess.Popen(cmd, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)

       for log in x.stdout.readlines():
          print '[ffmpeg info] {0}'.format(log)
       for log in x.stderr.readlines():
          print '[ffmpeg error] {0}'.format(log)

       print '\tSuccess!\n {0}  and {1} -> {2}\n'.format(folder + '/composeVideo.mp4', folder + '/composeAudio.mp3', folder +'/communicateVideo.mp4')

