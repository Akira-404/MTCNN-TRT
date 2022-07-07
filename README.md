# MTCNN TensorRT

## Usage

1. Set `TENSORRT_INCS` and `TENSORRT_LIBS` in "common/Makefile" correctly for your x86_64 system. More
   specifically, you should find the following lines in "common/Mafefile.config" and modify them if needed.

   ```
   # These are the directories where I installed TensorRT on my x86_64 PC.
   TENSORRT_INCS=-I"/usr/local/TensorRT-7.1.3.4/include"
   TENSORRT_LIBS=-L"/usr/local/TensorRT-7.1.3.4/lib"
   ```

2. Set `library_dirs` and `include_dirs` in "setup.py". More specifically, you should check and make sure the 2 TensorRT
   path lines are correct.

   ```python
   library_dirs = [
       '/usr/local/cuda/lib64',
       '/usr/local/TensorRT-7.1.3.4/lib',  # for my x86_64 PC
       '/usr/local/lib',
   ]
   ...
   include_dirs = [
       # in case the following numpy include path does not work, you
       # could replace it manually with, say,
       # '-I/usr/local/lib/python3.6/dist-packages/numpy/core/include',
       '-I' + numpy.__path__[0] + '/core/include',
       '-I/usr/local/cuda/include',
       '-I/usr/local/TensorRT-7.1.3.4/include',  # for my x86_64 PC
       '-I/usr/local/include',
   ]
   ```
1. Build the TensorRT engines from the pre-trained MTCNN model.  (Refer to [mtcnn/README.md](https://github.com/jkjung-avt/tensorrt_demos/blob/master/mtcnn/README.md) for more information about the prototxt and caffemodel files.)

   ```shell
   $ cd ${HOME}/project/tensorrt_demos/mtcnn
   $ make
   $ ./create_engines
   ```

2. Build the Cython code if it has not been done yet.  Refer to step 3 in Demo #1.

3. Run the "trt_mtcnn.py" demo program.  For example, I grabbed from the internet a poster of The Avengers for testing.

   ```shell
   $ cd ${HOME}/project/tensorrt_demos
   $ python3 trt_mtcnn.py --image ${HOME}/Pictures/avengers.jpg
   $ python3 trt_mtcnn.py --video ${HOME}/video/video.mp4
   ```

   Here's the result (JetPack-4.2.2, i.e. TensorRT 5).

   ![Avengers faces detected](https://raw.githubusercontent.com/jkjung-avt/tensorrt_demos/master/doc/avengers.png)

4. The "trt_mtcnn.py" demo program could also take various image inputs.  Refer to step 5 in Demo #1 for details.



