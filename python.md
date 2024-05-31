# Building python from source (courtesy of @kushalkolar)

## Step 1: Clone cpython
```
git clone https://github.com/python/cpython.git

cd cpython
```

## Step 2: Install
```
./configure --enable-optimizations
make -j7 # replace 7 with number of threads
sudo make altinstall
```
