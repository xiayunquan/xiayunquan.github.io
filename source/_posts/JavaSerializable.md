---
title: 序列化-Serializable和Parcelable的简单介绍
date: 2021-09-24 17:45:02
categories: Java
tags: 序列化
---

# 序列化的本质

序列化是一种用来处理对象流的机制。序列化是为了解决在对对象流进行读写操作时所引发的问题。

序列化：将java对象转换成字节序列的过程，字节码可以保存到数据库、内存、文件等，也可用于网络传输

反序列化：将字节序列恢复为java对象的过程。

序列化实现的方式有很多方案，在java中是使用的JDK内置的Serializable接口来实现序列化，而Android SDK中增加Parcelable方式来实现序列化，除了常见的这2种还有很多其他优秀的序列化和反序列化方案（Twitter的Serial、Google的Protocol Buffers和flatbuffers等），这里先了解一下Serializable和Parcelable2种方式的原理和区别。

# Serializable

Serializable是一个空的标记接口，没有任何方法和属性，implement Serializable只用于标记该对象是可以序列化的。如果一个类implement Serializable，则子类也是可以序列化的。

```java
/**
 * Serializability of a class is enabled by the class implementing the
 * java.io.Serializable interface. Classes that do not implement this
 * interface will not have any of their state serialized or
 * deserialized.  All subtypes of a serializable class are themselves
 * serializable.  The serialization interface has no methods or fields
 * and serves only to identify the semantics of being serializable. <p>
 *
 * To allow subtypes of non-serializable classes to be serialized, the
 * subtype may assume responsibility for saving and restoring the
 * state of the supertype's public, protected, and (if accessible)
 * package fields.  The subtype may assume this responsibility only if
 * the class it extends has an accessible no-arg constructor to
 * initialize the class's state.  It is an error to declare a class
 * Serializable if this is not the case.  The error will be detected at
 * runtime. <p>
 *
 * During deserialization, the fields of non-serializable classes will
 * be initialized using the public or protected no-arg constructor of
 * the class.  A no-arg constructor must be accessible to the subclass
 * that is serializable.  The fields of serializable subclasses will
 * be restored from the stream. <p>
 *
 * When traversing a graph, an object may be encountered that does not
 * support the Serializable interface. In this case the
 * NotSerializableException will be thrown and will identify the class
 * of the non-serializable object. <p>
 *
 * Classes that require special handling during the serialization and
 * deserialization process must implement special methods with these exact
 * signatures:
 *
 * <PRE>
 * private void writeObject(java.io.ObjectOutputStream out)
 *     throws IOException
 * private void readObject(java.io.ObjectInputStream in)
 *     throws IOException, ClassNotFoundException;
 * private void readObjectNoData()
 *     throws ObjectStreamException;
 * </PRE>
 *
 * <p>The writeObject method is responsible for writing the state of the
 * object for its particular class so that the corresponding
 * readObject method can restore it.  The default mechanism for saving
 * the Object's fields can be invoked by calling
 * out.defaultWriteObject. The method does not need to concern
 * itself with the state belonging to its superclasses or subclasses.
 * State is saved by writing the individual fields to the
 * ObjectOutputStream using the writeObject method or by using the
 * methods for primitive data types supported by DataOutput.
 *
 * <p>The readObject method is responsible for reading from the stream and
 * restoring the classes fields. It may call in.defaultReadObject to invoke
 * the default mechanism for restoring the object's non-static and
 * non-transient fields.  The defaultReadObject method uses information in
 * the stream to assign the fields of the object saved in the stream with the
 * correspondingly named fields in the current object.  This handles the case
 * when the class has evolved to add new fields. The method does not need to
 * concern itself with the state belonging to its superclasses or subclasses.
 * State is saved by writing the individual fields to the
 * ObjectOutputStream using the writeObject method or by using the
 * methods for primitive data types supported by DataOutput.
 *
 * <p>The readObjectNoData method is responsible for initializing the state of
 * the object for its particular class in the event that the serialization
 * stream does not list the given class as a superclass of the object being
 * deserialized.  This may occur in cases where the receiving party uses a
 * different version of the deserialized instance's class than the sending
 * party, and the receiver's version extends classes that are not extended by
 * the sender's version.  This may also occur if the serialization stream has
 * been tampered; hence, readObjectNoData is useful for initializing
 * deserialized objects properly despite a "hostile" or incomplete source
 * stream.
 *
 * <p>Serializable classes that need to designate an alternative object to be
 * used when writing an object to the stream should implement this
 * special method with the exact signature:
 *
 * <PRE>
 * ANY-ACCESS-MODIFIER Object writeReplace() throws ObjectStreamException;
 * </PRE><p>
 *
 * This writeReplace method is invoked by serialization if the method
 * exists and it would be accessible from a method defined within the
 * class of the object being serialized. Thus, the method can have private,
 * protected and package-private access. Subclass access to this method
 * follows java accessibility rules. <p>
 *
 * Classes that need to designate a replacement when an instance of it
 * is read from the stream should implement this special method with the
 * exact signature.
 *
 * <PRE>
 * ANY-ACCESS-MODIFIER Object readResolve() throws ObjectStreamException;
 * </PRE><p>
 *
 * This readResolve method follows the same invocation rules and
 * accessibility rules as writeReplace.<p>
 *
 * The serialization runtime associates with each serializable class a version
 * number, called a serialVersionUID, which is used during deserialization to
 * verify that the sender and receiver of a serialized object have loaded
 * classes for that object that are compatible with respect to serialization.
 * If the receiver has loaded a class for the object that has a different
 * serialVersionUID than that of the corresponding sender's class, then
 * deserialization will result in an {@link InvalidClassException}.  A
 * serializable class can declare its own serialVersionUID explicitly by
 * declaring a field named <code>"serialVersionUID"</code> that must be static,
 * final, and of type <code>long</code>:
 *
 * <PRE>
 * ANY-ACCESS-MODIFIER static final long serialVersionUID = 42L;
 * </PRE>
 *
 * If a serializable class does not explicitly declare a serialVersionUID, then
 * the serialization runtime will calculate a default serialVersionUID value
 * for that class based on various aspects of the class, as described in the
 * Java(TM) Object Serialization Specification.  However, it is <em>strongly
 * recommended</em> that all serializable classes explicitly declare
 * serialVersionUID values, since the default serialVersionUID computation is
 * highly sensitive to class details that may vary depending on compiler
 * implementations, and can thus result in unexpected
 * <code>InvalidClassException</code>s during deserialization.  Therefore, to
 * guarantee a consistent serialVersionUID value across different java compiler
 * implementations, a serializable class must declare an explicit
 * serialVersionUID value.  It is also strongly advised that explicit
 * serialVersionUID declarations use the <code>private</code> modifier where
 * possible, since such declarations apply only to the immediately declaring
 * class--serialVersionUID fields are not useful as inherited members. Array
 * classes cannot declare an explicit serialVersionUID, so they always have
 * the default computed value, but the requirement for matching
 * serialVersionUID values is waived for array classes.
 *
 * Android implementation of serialVersionUID computation will change slightly
 * for some classes if you're targeting android N. In order to preserve compatibility,
 * this change is only enabled is the application target SDK version is set to
 * 24 or higher. It is highly recommended to use an explicit serialVersionUID
 * field to avoid compatibility issues.
 *
 * <h3>Implement Serializable Judiciously</h3>
 * Refer to <i>Effective Java</i>'s chapter on serialization for thorough
 * coverage of the serialization API. The book explains how to use this
 * interface without harming your application's maintainability.
 *
 * <h3>Recommended Alternatives</h3>
 * <strong>JSON</strong> is concise, human-readable and efficient. Android
 * includes both a {@link android.util.JsonReader streaming API} and a {@link
 * org.json.JSONObject tree API} to read and write JSON. Use a binding library
 * like <a href="http://code.google.com/p/google-gson/">GSON</a> to read and
 * write Java objects directly.
 *
 * @author  unascribed
 * @see java.io.ObjectOutputStream
 * @see java.io.ObjectInputStream
 * @see java.io.ObjectOutput
 * @see java.io.ObjectInput
 * @see java.io.Externalizable
 * @since   JDK1.1
 */
public interface Serializable {
}
```

上面就是Serializable接口，可以看到注释特别长，总结一下大致如下：

1. Serializable接口没有方法和属性字段，用于标记类可以序列化。
2. 父类可序列化，则子类也可序列化。
3. static或用transient关键字标记的字段不会被序列化
4. Serializable内部是使用的ObjectOutputStream和ObjectInputStream来序列化和反序列化的，序列化时通过ObjectOutputStream的writeObject写入对象序列化流数据及状态，默认的保存机制是调用out.defaultWriteObject；反序列化时使用ObjectInputStream.readObject方法将流数据还原成类对象。
5. 序列化运行时是通过serialVersionUID来判断版本的一致性，我们可以在要序列化的类中显式的声明一个static final long serialVerisonUID值，默认情况下java编译器会自动给我们生成一个serialVersionUID，但由于不同的java编译器可能生成的serialVersionUID不同，反序列化期间可能导致InvalidClassException，所以强烈建议我们自己定义serialVersionUID值。
6. Serializable在使用是会产生大量临时变量，频繁GC，使用了反射，序列化过程较慢，所以官方推荐使用简洁高效的JSON代替Serializable。

### 序列化过程

下面看看ObjectOutputStream的writeObject相关的源码，简单看看序列化的过程

```java
/**
 * 外部调用，序列化入口
 */
public final void writeObject(Object obj) throws IOException {
	...
	writeObject0(obj, false);
	...
}

/**
 * 写数据，unshared=false
 */
private void writeObject0(Object obj, boolean unshared) throws IOException {
    ...
    //判断序列化对象的类型
    if (obj instanceof Class) {
        writeClass((Class) obj, unshared);
    } else if (obj instanceof ObjectStreamClass) {
        writeClassDesc((ObjectStreamClass) obj, unshared);
    // END Android-changed:  Make Class and ObjectStreamClass replaceable.
    } else if (obj instanceof String) {
        writeString((String) obj, unshared);
    } else if (cl.isArray()) {
        writeArray(obj, desc, unshared);
    } else if (obj instanceof Enum) {
        writeEnum((Enum<?>) obj, desc, unshared);
    } else if (obj instanceof Serializable) {
      	//如果是可序列化的，开始写入数据
        writeOrdinaryObject(obj, desc, unshared);
    } else {
      	//不可序列化，抛出异常
        if (extendedDebugInfo) {
            throw new NotSerializableException(
                cl.getName() + "\n" + debugInfoStack.toString());
        } else {
            throw new NotSerializableException(cl.getName());
        }
    }
    ...
}

/**
 * 写object数据
 */
private void writeOrdinaryObject(Object obj, ObjectStreamClass desc, boolean unshared) throws IOException {
    if (extendedDebugInfo) {
        debugInfoStack.push(
            (depth == 1 ? "root " : "") + "object (class \"" +
            obj.getClass().getName() + "\", " + obj.toString() + ")");
    }
    try {
        desc.checkSerialize();
      	//写入标记类型，读的时候根据这个来判断
        bout.writeByte(TC_OBJECT);
      	//写入类的描述信息
        writeClassDesc(desc, false);
        handles.assign(unshared ? null : obj);
        if (desc.isExternalizable() && !desc.isProxy()) {
          	//如果是实现了Externalizable接口，则写入ExternalData(
            writeExternalData((Externalizable) obj);
        } else {
          	//写序列化数据
            writeSerialData(obj, desc);
        }
    } finally {
        if (extendedDebugInfo) {
            debugInfoStack.pop();
        }
    }
}

/**
 * 真正写object data的入口
 */
private void writeSerialData(Object obj, ObjectStreamClass desc)
        throws IOException {
    ObjectStreamClass.ClassDataSlot[] slots = desc.getClassDataLayout();
  	//遍历父类和自己，for循环从父类开始写入数据
    for (int i = 0; i < slots.length; i++) {
        ObjectStreamClass slotDesc = slots[i].desc;
        if (slotDesc.hasWriteObjectMethod()) {
            PutFieldImpl oldPut = curPut;
            curPut = null;
            SerialCallbackContext oldContext = curContext;

            if (extendedDebugInfo) {
                debugInfoStack.push(
                    "custom writeObject data (class \"" +
                    slotDesc.getName() + "\")");
            }
            try {
                curContext = new SerialCallbackContext(obj, slotDesc);
                bout.setBlockDataMode(true);
                slotDesc.invokeWriteObject(obj, this);
                bout.setBlockDataMode(false);
                bout.writeByte(TC_ENDBLOCKDATA);
            } finally {
                curContext.setUsed();
                curContext = oldContext;
                if (extendedDebugInfo) {
                    debugInfoStack.pop();
                }
            }

            curPut = oldPut;
        } else {
          	//写入字段属性值
            defaultWriteFields(obj, slotDesc);
        }
    }
}

/**
 * 写基本数据类型数据
 */
private void defaultWriteFields(Object obj, ObjectStreamClass desc)
        throws IOException {
    Class<?> cl = desc.forClass();
    if (cl != null && obj != null && !cl.isInstance(obj)) {
        throw new ClassCastException();
    }

    desc.checkDefaultSerialize();

    int primDataSize = desc.getPrimDataSize();
    if (primVals == null || primVals.length < primDataSize) {
        primVals = new byte[primDataSize];
    }
    desc.getPrimFieldValues(obj, primVals);
    bout.write(primVals, 0, primDataSize, false);

    ObjectStreamField[] fields = desc.getFields(false);
    Object[] objVals = new Object[desc.getNumObjFields()];
    int numPrimFields = fields.length - objVals.length;
    desc.getObjFieldValues(obj, objVals);
    for (int i = 0; i < objVals.length; i++) {
        if (extendedDebugInfo) {
            debugInfoStack.push(
                "field (class \"" + desc.getName() + "\", name: \"" +
                fields[numPrimFields + i].getName() + "\", type: \"" +
                fields[numPrimFields + i].getType() + "\")");
        }
        try {
            writeObject0(objVals[i],
                         fields[numPrimFields + i].isUnshared());
        } finally {
            if (extendedDebugInfo) {
                debugInfoStack.pop();
            }
        }
    }
}
```

### 反序列化过程

再看看ObjectInputStream的readObject相关的源码，反序列化的过程

```java
public final Object readObject() throws IOException, ClassNotFoundException {
    ...
    Object obj = readObject0(false);
    ...
}

/**
 * 读object数据
 */
private Object readObject0(boolean unshared) throws IOException {
    ...
    byte tc;
    while ((tc = bin.peekByte()) == TC_RESET) {
        bin.readByte();
        handleReset();
    }

    depth++;
    // Android-removed: ObjectInputFilter logic, to be reconsidered. http://b/110252929
    // totalObjectRefs++;
    try {
        switch (tc) {
            case TC_NULL:
                return readNull();

            case TC_REFERENCE:
                return readHandle(unshared);

            case TC_CLASS:
                return readClass(unshared);
            
            case TC_CLASSDESC:
            case TC_PROXYCLASSDESC:
                return readClassDesc(unshared);

            case TC_STRING:
            case TC_LONGSTRING:
                return checkResolve(readString(unshared));

            case TC_ARRAY:
                return checkResolve(readArray(unshared));

            case TC_ENUM:
                return checkResolve(readEnum(unshared));
            
            //判断类型如果是object类型，则读取object对象流
            case TC_OBJECT:
                return checkResolve(readOrdinaryObject(unshared));

            case TC_EXCEPTION:
                IOException ex = readFatalException();
                throw new WriteAbortedException("writing aborted", ex);

            case TC_BLOCKDATA:
            case TC_BLOCKDATALONG:
                if (oldMode) {
                    bin.setBlockDataMode(true);
                    bin.peek();             // force header read
                    throw new OptionalDataException(
                        bin.currentBlockRemaining());
                } else {
                    throw new StreamCorruptedException(
                        "unexpected block data");
                }

            case TC_ENDBLOCKDATA:
                if (oldMode) {
                    throw new OptionalDataException(true);
                } else {
                    throw new StreamCorruptedException(
                        "unexpected end of block data");
                }

            default:
                throw new StreamCorruptedException(
                    String.format("invalid type code: %02X", tc));
        }
    } finally {
        depth--;
        bin.setBlockDataMode(oldMode);
    }
}

/**
 * 读取流数据
 */
private Object readOrdinaryObject(boolean unshared) throws IOException {
    if (bin.readByte() != TC_OBJECT) {
        throw new InternalError();
    }

    ObjectStreamClass desc = readClassDesc(false);
    desc.checkDeserialize();

    Class<?> cl = desc.forClass();
    if (cl == String.class || cl == Class.class
            || cl == ObjectStreamClass.class) {
        throw new InvalidClassException("invalid class descriptor");
    }

    Object obj;
    try {
        obj = desc.isInstantiable() ? desc.newInstance() : null;
    } catch (Exception ex) {
        throw (IOException) new InvalidClassException(
            desc.forClass().getName(),
            "unable to create instance").initCause(ex);
    }

    passHandle = handles.assign(unshared ? unsharedMarker : obj);
    ClassNotFoundException resolveEx = desc.getResolveException();
    if (resolveEx != null) {
        handles.markException(passHandle, resolveEx);
    }

    if (desc.isExternalizable()) {
      	//读取自定义序列化数据
        readExternalData((Externalizable) obj, desc);
    } else {
      	//读取序列化数据
        readSerialData(obj, desc);
    }
    ...
    return obj;
}

/**
 * 读取序列化数据
 */
private void readSerialData(Object obj, ObjectStreamClass desc) throws IOException {
    ObjectStreamClass.ClassDataSlot[] slots = desc.getClassDataLayout();
    for (int i = 0; i < slots.length; i++) {
        ObjectStreamClass slotDesc = slots[i].desc;

        if (slots[i].hasData) {
            if (obj == null || handles.lookupException(passHandle) != null) {
                defaultReadFields(null, slotDesc); // skip field values
            } else if (slotDesc.hasReadObjectMethod()) {
                // BEGIN Android-changed: ThreadDeath cannot cause corruption on Android.
                // Android does not support Thread.stop() or Thread.stop(Throwable) so this
                // does not need to protect against state corruption that can occur when a
                // ThreadDeath Error is thrown in the middle of the finally block.
                SerialCallbackContext oldContext = curContext;
                if (oldContext != null)
                    oldContext.check();
                try {
                    curContext = new SerialCallbackContext(obj, slotDesc);

                    bin.setBlockDataMode(true);
                    slotDesc.invokeReadObject(obj, this);
                } catch (ClassNotFoundException ex) {
                    /*
                     * In most cases, the handle table has already
                     * propagated a CNFException to passHandle at this
                     * point; this mark call is included to address cases
                     * where the custom readObject method has cons'ed and
                     * thrown a new CNFException of its own.
                     */
                    handles.markException(passHandle, ex);
                } finally {
                    curContext.setUsed();
                    if (oldContext!= null)
                        oldContext.check();
                    curContext = oldContext;
                    // END Android-changed: ThreadDeath cannot cause corruption on Android.
                }

                /*
                 * defaultDataEnd may have been set indirectly by custom
                 * readObject() method when calling defaultReadObject() or
                 * readFields(); clear it to restore normal read behavior.
                 */
                defaultDataEnd = false;
            } else {
                defaultReadFields(obj, slotDesc);
            }

            if (slotDesc.hasWriteObjectData()) {
                skipCustomData();
            } else {
                bin.setBlockDataMode(false);
            }
        } else {
            if (obj != null &&
                slotDesc.hasReadObjectNoDataMethod() &&
                handles.lookupException(passHandle) == null)
            {
                slotDesc.invokeReadObjectNoData(obj);
            }
        }
    }
}

/**
 * 读取字段属性数据
 */
private void defaultReadFields(Object obj, ObjectStreamClass desc) throws IOException {
    Class<?> cl = desc.forClass();
    if (cl != null && obj != null && !cl.isInstance(obj)) {
        throw new ClassCastException();
    }

    int primDataSize = desc.getPrimDataSize();
    if (primVals == null || primVals.length < primDataSize) {
        primVals = new byte[primDataSize];
    }
        bin.readFully(primVals, 0, primDataSize, false);
    if (obj != null) {
        desc.setPrimFieldValues(obj, primVals);
    }

    int objHandle = passHandle;
    ObjectStreamField[] fields = desc.getFields(false);
    Object[] objVals = new Object[desc.getNumObjFields()];
    int numPrimFields = fields.length - objVals.length;
    for (int i = 0; i < objVals.length; i++) {
        ObjectStreamField f = fields[numPrimFields + i];
        objVals[i] = readObject0(f.isUnshared());
        if (f.getField() != null) {
            handles.markDependency(objHandle, passHandle);
        }
    }
    if (obj != null) {
        desc.setObjFieldValues(obj, objVals);
    }
    passHandle = objHandle;
}
```

可以看到ObjectInputStream反序列化和ObjectOutputStream方法是一一对应，怎么写就怎么读取，其实java io API内部设计都是这样，所有的Input和Output，Reader和Writer方法基本都是一一对应，掌握了输出流就知道输入流对应该怎么写。

### Externalizable

从上面的源码中可以看到有一个这个判断：

```java
//ObjectOutputStream
if (desc.isExternalizable() && !desc.isProxy()) {
	writeExternalData((Externalizable) obj);
} 

//ObjectInputStream
if (desc.isExternalizable()) {
	readExternalData((Externalizable) obj, desc);
}
```

Serializable有一个直接子类Externalizable接口，该接口有两个方法`writeExternal(ObjectOutput out)`、`readExternal(ObjectInput in)`，可以使用该接口来实现自定义序列化细节。看下源码

```java

/**
 * Only the identity of the class of an Externalizable instance is
 * written in the serialization stream and it is the responsibility
 * of the class to save and restore the contents of its instances.
 *
 * The writeExternal and readExternal methods of the Externalizable
 * interface are implemented by a class to give the class complete
 * control over the format and contents of the stream for an object
 * and its supertypes. These methods must explicitly
 * coordinate with the supertype to save its state. These methods supersede
 * customized implementations of writeObject and readObject methods.<br>
 *
 * Object Serialization uses the Serializable and Externalizable
 * interfaces.  Object persistence mechanisms can use them as well.  Each
 * object to be stored is tested for the Externalizable interface. If
 * the object supports Externalizable, the writeExternal method is called. If the
 * object does not support Externalizable and does implement
 * Serializable, the object is saved using
 * ObjectOutputStream. <br> When an Externalizable object is
 * reconstructed, an instance is created using the public no-arg
 * constructor, then the readExternal method called.  Serializable
 * objects are restored by reading them from an ObjectInputStream.<br>
 *
 * An Externalizable instance can designate a substitution object via
 * the writeReplace and readResolve methods documented in the Serializable
 * interface.<br>
 *
 * @author  unascribed
 * @see java.io.ObjectOutputStream
 * @see java.io.ObjectInputStream
 * @see java.io.ObjectOutput
 * @see java.io.ObjectInput
 * @see java.io.Serializable
 * @since   JDK1.1
 */
public interface Externalizable extends java.io.Serializable {
    /**
     * The object implements the writeExternal method to save its contents
     * by calling the methods of DataOutput for its primitive values or
     * calling the writeObject method of ObjectOutput for objects, strings,
     * and arrays.
     *
     * @serialData Overriding methods should use this tag to describe
     *             the data layout of this Externalizable object.
     *             List the sequence of element types and, if possible,
     *             relate the element to a public/protected field and/or
     *             method of this Externalizable class.
     *
     * @param out the stream to write the object to
     * @exception IOException Includes any I/O exceptions that may occur
     */
    void writeExternal(ObjectOutput out) throws IOException;

    /**
     * The object implements the readExternal method to restore its
     * contents by calling the methods of DataInput for primitive
     * types and readObject for objects, strings and arrays.  The
     * readExternal method must read the values in the same sequence
     * and with the same types as were written by writeExternal.
     *
     * @param in the stream to read data from in order to restore the object
     * @exception IOException if I/O errors occur
     * @exception ClassNotFoundException If the class for an object being
     *              restored cannot be found.
     */
    void readExternal(ObjectInput in) throws IOException, ClassNotFoundException;
}
```

看一个示例，android.telephony.mbms.DownloadRequest中的SerializationDataContainer内部类实现了Externalizable接口来自定义序列化内容，代码如下。

```java
private static class SerializationDataContainer implements Externalizable {
    private String fileServiceId;
    private Uri source;
    private Uri destination;
    private int subscriptionId;
    private String appIntent;
    private int version;

    public SerializationDataContainer() {}

    SerializationDataContainer(DownloadRequest request) {
        fileServiceId = request.fileServiceId;
        source = request.sourceUri;
        destination = request.destinationUri;
        subscriptionId = request.subscriptionId;
        appIntent = request.serializedResultIntentForApp;
        version = request.version;
    }

    @Override
    public void writeExternal(ObjectOutput objectOutput) throws IOException {
        objectOutput.write(version);
        objectOutput.writeUTF(fileServiceId);
        objectOutput.writeUTF(source.toString());
        objectOutput.writeUTF(destination.toString());
        objectOutput.write(subscriptionId);
        objectOutput.writeUTF(appIntent);
    }

    @Override
    public void readExternal(ObjectInput objectInput) throws IOException {
        version = objectInput.read();
        fileServiceId = objectInput.readUTF();
        source = Uri.parse(objectInput.readUTF());
        destination = Uri.parse(objectInput.readUTF());
        subscriptionId = objectInput.read();
        appIntent = objectInput.readUTF();
        // Do version checks here -- future versions may have other fields.
    }
}

//写入序列化数据
public byte[] toByteArray() {
    try {
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        ObjectOutputStream stream = new ObjectOutputStream(byteArrayOutputStream);
        SerializationDataContainer container = new SerializationDataContainer(this);
        stream.writeObject(container);
        stream.flush();
        return byteArrayOutputStream.toByteArray();
    } catch (IOException e) {
        // Really should never happen
        Log.e(LOG_TAG, "Got IOException trying to serialize opaque data");
        return null;
    }
}

//还原序列化对象
public static Builder fromSerializedRequest(byte[] data) {
    Builder builder;
    try {
        ObjectInputStream stream = new ObjectInputStream(new ByteArrayInputStream(data));
        SerializationDataContainer dataContainer =
                (SerializationDataContainer) stream.readObject();
        builder = new Builder(dataContainer.source, dataContainer.destination);
        builder.version = dataContainer.version;
        builder.appIntent = dataContainer.appIntent;
        builder.fileServiceId = dataContainer.fileServiceId;
        builder.subscriptionId = dataContainer.subscriptionId;
    } catch (IOException e) {
        // Really should never happen
        Log.e(LOG_TAG, "Got IOException trying to parse opaque data");
        throw new IllegalArgumentException(e);
    } catch (ClassNotFoundException e) {
        Log.e(LOG_TAG, "Got ClassNotFoundException trying to parse opaque data");
        throw new IllegalArgumentException(e);
    }
    return builder;
}
```

到此Serializable了解的差不多了，下面看下Android中的Parcelable实现序列化的一些细节。



# Parcelable

Parcelable是Android中的一种序列化机制，Parcelable的原理是通过Parcel将一个完整的对象进行分解，而分解后的每一部分都是Intent所支持的数据类型，Parcel机制是将序列化的数据写入到一个共享内存当中，其他进程可以通过Parcel从这块共享内存中读出字节流，因此可以实现在Binder中跨进程传输数据，实现传递对象的功能。

看下Parcelable接口的源码

```java
package android.os;

import android.annotation.IntDef;

import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

/**
 * Interface for classes whose instances can be written to
 * and restored from a {@link Parcel}.  Classes implementing the Parcelable
 * interface must also have a non-null static field called <code>CREATOR</code>
 * of a type that implements the {@link Parcelable.Creator} interface.
 * 
 * <p>A typical implementation of Parcelable is:</p>
 * 
 * <pre>
 * public class MyParcelable implements Parcelable {
 *     private int mData;
 *
 *     public int describeContents() {
 *         return 0;
 *     }
 *
 *     public void writeToParcel(Parcel out, int flags) {
 *         out.writeInt(mData);
 *     }
 *
 *     public static final Parcelable.Creator&lt;MyParcelable&gt; CREATOR
 *             = new Parcelable.Creator&lt;MyParcelable&gt;() {
 *         public MyParcelable createFromParcel(Parcel in) {
 *             return new MyParcelable(in);
 *         }
 *
 *         public MyParcelable[] newArray(int size) {
 *             return new MyParcelable[size];
 *         }
 *     };
 *     
 *     private MyParcelable(Parcel in) {
 *         mData = in.readInt();
 *     }
 * }</pre>
 */
public interface Parcelable {
    /** @hide */
    @IntDef(flag = true, prefix = { "PARCELABLE_" }, value = {
            PARCELABLE_WRITE_RETURN_VALUE,
            PARCELABLE_ELIDE_DUPLICATES,
    })
    @Retention(RetentionPolicy.SOURCE)
    public @interface WriteFlags {}

    /**
     * Flag for use with {@link #writeToParcel}: the object being written
     * is a return value, that is the result of a function such as
     * "<code>Parcelable someFunction()</code>",
     * "<code>void someFunction(out Parcelable)</code>", or
     * "<code>void someFunction(inout Parcelable)</code>".  Some implementations
     * may want to release resources at this point.
     */
    public static final int PARCELABLE_WRITE_RETURN_VALUE = 0x0001;

    /**
     * Flag for use with {@link #writeToParcel}: a parent object will take
     * care of managing duplicate state/data that is nominally replicated
     * across its inner data members.  This flag instructs the inner data
     * types to omit that data during marshaling.  Exact behavior may vary
     * on a case by case basis.
     * @hide
     */
    public static final int PARCELABLE_ELIDE_DUPLICATES = 0x0002;

    /*
     * Bit masks for use with {@link #describeContents}: each bit represents a
     * kind of object considered to have potential special significance when
     * marshalled.
     */

    /** @hide */
    @IntDef(flag = true, prefix = { "CONTENTS_" }, value = {
            CONTENTS_FILE_DESCRIPTOR,
    })
    @Retention(RetentionPolicy.SOURCE)
    public @interface ContentsFlags {}

    /**
     * Descriptor bit used with {@link #describeContents()}: indicates that
     * the Parcelable object's flattened representation includes a file descriptor.
     *
     * @see #describeContents()
     */
    public static final int CONTENTS_FILE_DESCRIPTOR = 0x0001;
    
    /**
     * Describe the kinds of special objects contained in this Parcelable
     * instance's marshaled representation. For example, if the object will
     * include a file descriptor in the output of {@link #writeToParcel(Parcel, int)},
     * the return value of this method must include the
     * {@link #CONTENTS_FILE_DESCRIPTOR} bit.
     *  
     * @return a bitmask indicating the set of special object types marshaled
     * by this Parcelable object instance.
     */
    public @ContentsFlags int describeContents();
    
    /**
     * Flatten this object in to a Parcel.
     * 
     * @param dest The Parcel in which the object should be written.
     * @param flags Additional flags about how the object should be written.
     * May be 0 or {@link #PARCELABLE_WRITE_RETURN_VALUE}.
     */
    public void writeToParcel(Parcel dest, @WriteFlags int flags);

    /**
     * Interface that must be implemented and provided as a public CREATOR
     * field that generates instances of your Parcelable class from a Parcel.
     */
    public interface Creator<T> {
        /**
         * Create a new instance of the Parcelable class, instantiating it
         * from the given Parcel whose data had previously been written by
         * {@link Parcelable#writeToParcel Parcelable.writeToParcel()}.
         * 
         * @param source The Parcel to read the object's data from.
         * @return Returns a new instance of the Parcelable class.
         */
        public T createFromParcel(Parcel source);
        
        /**
         * Create a new array of the Parcelable class.
         * 
         * @param size Size of the array.
         * @return Returns an array of the Parcelable class, with every entry
         * initialized to null.
         */
        public T[] newArray(int size);
    }

    /**
     * Specialization of {@link Creator} that allows you to receive the
     * ClassLoader the object is being created in.
     */
    public interface ClassLoaderCreator<T> extends Creator<T> {
        /**
         * Create a new instance of the Parcelable class, instantiating it
         * from the given Parcel whose data had previously been written by
         * {@link Parcelable#writeToParcel Parcelable.writeToParcel()} and
         * using the given ClassLoader.
         *
         * @param source The Parcel to read the object's data from.
         * @param loader The ClassLoader that this object is being created in.
         * @return Returns a new instance of the Parcelable class.
         */
        public T createFromParcel(Parcel source, ClassLoader loader);
    }
}
```

使用Parcelable接口实现序列化需要：

1. 创建一个实现了Parcelable.Creator接口的静态常量，其中createFromParcel方法用于反序列化时创建新的实例。

2. 重写`writeToParcel(Parcel out, int flags)` 方法通过Parcel将数据写入共享内存，parcel可以写入原始数据类型（如：writeInt()、writeFloat()、writeString()等）、也可以写入一个Parcelable对象`writeParcelable(Parcelable p, int parcelableFlags)`、还提供了2种写入序列化对象集合的方式`writeList(List)`、`writeTypedList(List)`
3. 声明一个Parcel参数的构造方法读取Parcel数据

Parcel中的写和读都是native方法，底层采用c语言编写，具体的源码就不去研究了。

在内存上序列化的时候，Parcelable比Serializable性能高，所以尽量使用Parcelable来序列化；但是Parcelable不适合磁盘上的数据持久化，所以这种情况使用Serializable更好。