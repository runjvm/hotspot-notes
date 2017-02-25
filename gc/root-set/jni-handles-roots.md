jobject, jstring, jclass etc. are used to represent Java objects in C/C++ code.

For example
```c++
jclass cls = env->FindClass("Main");
```

In HotSpot, jobject etc. are called jni handles, and are oop\*. They can be created by `jobject JNIHandleBlock::allocate_handle(oop obj)`, which returns a oop\*. During a GC when the oop pointed by a jni handle is updated, the native code would be using the new oop without modifying the native code. These are root oops because they record the states of native code.

In C++' jni.h, jobject etc. are defined as pointers to empty class.

```c++
#ifdef __cplusplus

class _jobject {};
class _jclass : public _jobject {};
class _jthrowable : public _jobject {};
class _jstring : public _jobject {};
class _jarray : public _jobject {};
class _jbooleanArray : public _jarray {};
class _jbyteArray : public _jarray {};
class _jcharArray : public _jarray {};
class _jshortArray : public _jarray {};
class _jintArray : public _jarray {};
class _jlongArray : public _jarray {};
class _jfloatArray : public _jarray {};
class _jdoubleArray : public _jarray {};
class _jobjectArray : public _jarray {};

typedef _jobject *jobject;
typedef _jclass *jclass;
typedef _jthrowable *jthrowable;
typedef _jstring *jstring;
typedef _jarray *jarray;
typedef _jbooleanArray *jbooleanArray;
typedef _jbyteArray *jbyteArray;
typedef _jcharArray *jcharArray;
typedef _jshortArray *jshortArray;
typedef _jintArray *jintArray;
typedef _jlongArray *jlongArray;
typedef _jfloatArray *jfloatArray;
typedef _jdoubleArray *jdoubleArray;
typedef _jobjectArray *jobjectArray;

```
