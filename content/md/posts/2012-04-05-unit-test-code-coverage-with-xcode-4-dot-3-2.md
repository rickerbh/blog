{:layout :post, :title "\"Unit Test Code Coverage with Xcode 4.3.2\"", :date "2012-04-05 13:11"}
I upgraded to Xcode 4.3.2 and this seemed to break unit tests on iOS for me.

The error that was being reported was:
    Test rig '/Applications/Xcode.app/.../iPhoneSimulator.platform/.../otest' exited abnormally with code 134 (it may have crashed).

(... used to save space)

The actual cause of this error was explained in the unit test build/run log files in Xcode:
    Detected an attempt to call a symbol in system libraries that is not present on the iPhone:
    fopen$UNIX2003 called from function llvm_gcda_start_file in image UnitTests.

I covered this issue in a prior post, but the short version of how to get around it is to include the below code in one of your test .m files, outside of the @implementation...@end block. I recommend right down the bottom of one of the files.

    #include <stdio.h>
    // Prototype declarations
    FILE *fopen$UNIX2003( const char *filename, const char *mode );
    size_t fwrite$UNIX2003( const void *a, size_t b, size_t c, FILE *d );

    FILE *fopen$UNIX2003( const char *filename, const char *mode ) {
      return fopen(filename, mode);
    }
    size_t fwrite$UNIX2003( const void *a, size_t b, size_t c, FILE *d ) {
      return fwrite(a, b, c, d);
    }

Other settings for successful execution of the unit tests with code coverage are (in your Project's Unit Test target):

* Generate Test Coverage Files = YES
* Instrument Program Flows = YES

There is no need for linking libprofile_rt to get coverage to work.
