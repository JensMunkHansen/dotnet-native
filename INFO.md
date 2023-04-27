%include cs_struct.i

cs_struct(CMyStruct, MyStruct)

 

struct CMyStruct {

          int x;

          double y;

          long z;

};

bool getMyData(CMyStruct &cms);

 

You must also define a C# equivalent of the structure called “struct MyStruct”, and compile this definition along with the SWIG-generated files.

 

If you would like to use a C# class instead (“class MyStruct”), use the following macro:

 

////////////////////////////////////////////////////////////////////////////////

// %cs_class(TYPE, CSTYPE)

//

// Maps a C++ struct or class to a C# class that you have written yourself.

// The class should have [StructLayout(LayoutKind.Sequential)] in front of it.

// This macro should be used rather than cs_struct if the structure in question

// is always passed by reference (because it is large). Double pointers and

// references (**, *&) are not supported here, though I think it's possible to

// do so. I suspect that the .NET marshaller copies the class both on input and

// output, so function calls involving it are probably slow.

//

// It seems that if you pass a class to a C++ function as a parameter, any

// changes made by the C++ function will NOT be copied back to C#. So if that's

// what you need, there are two workarounds:

// 1. Pass a C# struct and use %cs_struct instead.

// 2. Change the C++ function to return a pointer to the same parameter that

//    was passed in. Then your C# call will have to look like

//    "a = Klass.Function(a)". Of course, if the function already returns a

//    value then you're in a tough spot. Sorry.

%define %cs_class(TYPE, CSTYPE)

                %ignore TYPE;

                %typemap(ctype)    TYPE*, TYPE&               %{ TYPE* %}

                %typemap(in)       TYPE*, TYPE&               %{ $1 = $input; %}

                %typemap(varin)    TYPE*, TYPE&               %{ $1 = $input; %}

                //%typemap(memberin) TYPE*, TYPE&               %{ $1 = $input; %}

                %typemap(out, null="NULL") TYPE*, TYPE&       %{ $result = $1; %}

                %typemap(imtype, out="IntPtr") TYPE*, TYPE&   %{ CSTYPE %}

                %typemap(cstype)   TYPE*, TYPE&               %{ CSTYPE %}

                %typemap(csin)     TYPE*, TYPE&               %{ $csinput %}

                %typemap(csout, excode=SWIGEXCODE) TYPE*, TYPE& {

                                IntPtr ptr = $imcall;$excode

                                if (ptr == IntPtr.Zero)

                                                return null;

                                CSTYPE ret = (CSTYPE)Marshal.PtrToStructure(ptr, typeof(CSTYPE));

                                return ret;

                }

                %typemap(csvarin, excode=SWIGEXCODE2) TYPE*

                %{

                                set { $imcall;$excode }

                %}

                %typemap(csvarout, excode=SWIGEXCODE2) TYPE*

                %{

                                get {

                                                IntPtr ptr = $imcall;$excode

                                                if (ptr == IntPtr.Zero)

                                                                return null;

                                                CSTYPE ret = (CSTYPE)Marshal.PtrToStructure(ptr, typeof(CSTYPE));

                                                return ret;

                                }

                %}

%enddef
