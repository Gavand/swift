# -*- mode: cmake -*-

cmake_minimum_required(VERSION 2.8.12)

# Add path for custom CMake modules.
list(APPEND CMAKE_MODULE_PATH
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules")

# Load utility modules.
include(SwiftBenchmarkUtils)

set(SWIFT_BENCHMARK_BUILT_STANDALONE FALSE)
if("${CMAKE_SOURCE_DIR}" STREQUAL "${CMAKE_CURRENT_SOURCE_DIR}")
  set(SWIFT_BENCHMARK_BUILT_STANDALONE TRUE)
endif()

if(SWIFT_BENCHMARK_SUBCMAKE_BUILD)
  precondition(SWIFT_BENCHMARK_BUILT_STANDALONE
    MESSAGE "If we are a subcmake build, we must be built standalone")
endif()

include(AddSwiftBenchmarkSuite)

#===-----------------------------------------------------------------------===#
# Declarative Description of Benchmarks
#===-----------------------------------------------------------------------===#

set(SWIFT_BENCH_MODULES
    single-source/Ackermann
    single-source/AngryPhonebook
    single-source/AnyHashableWithAClass
    single-source/Array2D
    single-source/ArrayAppend
    single-source/ArrayInClass
    single-source/ArrayLiteral
    single-source/ArrayOfGenericPOD
    single-source/ArrayOfGenericRef
    single-source/ArrayOfPOD
    single-source/ArrayOfRef
    single-source/ArraySetElement
    single-source/ArraySubscript
    single-source/BinaryFloatingPointConversionFromBinaryInteger
    single-source/BinaryFloatingPointProperties
    single-source/BitCount
    single-source/ByteSwap
    single-source/COWTree
    single-source/COWArrayGuaranteedParameterOverhead
    single-source/CString
    single-source/CSVParsing
    single-source/Calculator
    single-source/CaptureProp
    single-source/ChainedFilterMap
    single-source/CharacterLiteralsLarge
    single-source/CharacterLiteralsSmall
    single-source/CharacterProperties
    single-source/Chars
    single-source/ClassArrayGetter
    single-source/Combos
    single-source/DataBenchmarks
    single-source/DeadArray
    single-source/DictOfArraysToArrayOfDicts
    single-source/DictTest
    single-source/DictTest2
    single-source/DictTest3
    single-source/DictTest4
    single-source/DictTest4Legacy
    single-source/DictionaryBridge
    single-source/DictionaryBridgeToObjC
    single-source/DictionaryCompactMapValues
    single-source/DictionaryCopy
    single-source/DictionaryGroup
    single-source/DictionaryKeysContains
    single-source/DictionaryLiteral
    single-source/DictionaryRemove
    single-source/DictionarySubscriptDefault
    single-source/DictionarySwap
    single-source/DoubleWidthDivision
    single-source/DropFirst
    single-source/DropLast
    single-source/DropWhile
    single-source/ErrorHandling
    single-source/Exclusivity
    single-source/ExistentialPerformance
    single-source/Fibonacci
    single-source/FloatingPointPrinting
    single-source/Hanoi
    single-source/Hash
    single-source/HashQuadratic
    single-source/Histogram
    single-source/Integrate
    single-source/IterateData
    single-source/Join
    single-source/LazyFilter
    single-source/LinkedList
    single-source/LuhnAlgoEager
    single-source/LuhnAlgoLazy
    single-source/MapReduce
    single-source/Memset
    single-source/MonteCarloE
    single-source/MonteCarloPi
    single-source/NSDictionaryCastToSwift
    single-source/NSError
    single-source/NSStringConversion
    single-source/NibbleSort
    single-source/NopDeinit
    single-source/ObjectAllocation
    single-source/ObjectiveCBridging
    single-source/ObjectiveCBridgingStubs
    single-source/ObjectiveCNoBridgingStubs
    single-source/ObserverClosure
    single-source/ObserverForwarderStruct
    single-source/ObserverPartiallyAppliedMethod
    single-source/ObserverUnappliedMethod
    single-source/OpaqueConsumingUsers
    single-source/OpenClose
    single-source/Phonebook
    single-source/PointerArithmetics
    single-source/PolymorphicCalls
    single-source/PopFront
    single-source/PopFrontGeneric
    single-source/Prefix
    single-source/PrefixWhile
    single-source/Prims
    single-source/ProtocolDispatch
    single-source/ProtocolDispatch2
    single-source/Queue
    single-source/RC4
    single-source/RGBHistogram
    single-source/Radix2CooleyTukey
    single-source/RandomShuffle
    single-source/RandomValues
    single-source/RangeAssignment
    single-source/RangeIteration
    single-source/RangeReplaceableCollectionPlusDefault
    single-source/RecursiveOwnedParameter
    single-source/ReduceInto
    single-source/RemoveWhere
    single-source/ReversedCollections
    single-source/RomanNumbers
    single-source/SequenceAlgos
    single-source/SetTests
    single-source/SevenBoom
    single-source/Sim2DArray
    single-source/SortLargeExistentials
    single-source/SortLettersInPlace
    single-source/SortStrings
    single-source/StackPromo
    single-source/StaticArray
    single-source/StrComplexWalk
    single-source/StrToInt
    single-source/StringBuilder
    single-source/StringComparison
    single-source/StringEdits
    single-source/StringEnum
    single-source/StringInterpolation
    single-source/StringMatch
    single-source/StringRemoveDupes
    single-source/StringTests
    single-source/StringWalk
    single-source/Substring
    single-source/Suffix
    single-source/SuperChars
    single-source/TwoSum
    single-source/TypeFlood
    single-source/UTF8Decode
    single-source/Walsh
    single-source/WordCount
    single-source/XorLoop
)

set(SWIFT_MULTISOURCE_SWIFT3_BENCHES
  multi-source/PrimsSplit
)

set(PrimsSplit_sources
    multi-source/PrimsSplit/Prims.swift
    multi-source/PrimsSplit/Prims_main.swift)

set(SWIFT_MULTISOURCE_SWIFT4_BENCHES
)

set(BENCH_DRIVER_LIBRARY_FLAGS)
if (SWIFT_RUNTIME_ENABLE_LEAK_CHECKER)
  set(BENCH_DRIVER_LIBRARY_FLAGS -DSWIFT_RUNTIME_ENABLE_LEAK_CHECKER)
endif()

set(BENCH_LIBRARY_MODULES
    utils/TestsUtils
)

set(BENCH_DRIVER_LIBRARY_MODULES
    utils/DriverUtils
)

#===-----------------------------------------------------------------------===#
# Build Configuration
#===-----------------------------------------------------------------------===#

# You have to delete CMakeCache.txt in the swift build to force a
# reconfiguration.
set(SWIFT_EXTRA_BENCH_CONFIGS CACHE STRING
    "A semicolon separated list of benchmark configurations. \
Available configurations: <Optlevel>_SINGLEFILE, <Optlevel>_MULTITHREADED")

# Syntax for an optset:  <optimization-level>_<configuration>
#    where "_<configuration>" is optional.
if(NOT SWIFT_OPTIMIZATION_LEVELS)
  set(SWIFT_OPTIMIZATION_LEVELS "Onone" "O" "Osize"
                                ${SWIFT_EXTRA_BENCH_CONFIGS})
endif()

set(SWIFT_BENCHMARK_EXTRA_FLAGS "" CACHE STRING
    "Extra options to pass to swiftc when building the benchmarks")

if (SWIFT_BENCHMARK_BUILT_STANDALONE)
  # This option's value must match the value of the same option used when
  # building the swift runtime.
  option(SWIFT_RUNTIME_ENABLE_LEAK_CHECKER
    "Should the runtime be built with support for non-thread-safe leak detecting entrypoints"
    FALSE)
endif()

set(SWIFT_BENCHMARK_NUM_O_ITERATIONS "" CACHE STRING
    "Number of iterations to perform when running -O benchmarks via cmake")
set(SWIFT_BENCHMARK_NUM_ONONE_ITERATIONS "" CACHE STRING
    "Number of iterations to perform when running -Onone benchmarks via cmake")

# Options for the default (= empty) configuration
set(BENCHOPTS "-whole-module-optimization")

option(SWIFT_BENCHMARK_GENERATE_OPT_VIEW
  "Produce optimization view for benchmarks"
  FALSE)

set(SWIFT_BENCHMARK_OPT_VIEWER "" CACHE FILEPATH
  "Path to opt-viewer")

if(SWIFT_BENCHMARK_OPT_VIEWER)
  # If the path to the opt-viewer was specified manually and we have no access
  # to the LLVM tree, assume we have the modules for the opt-viewer installed.
  if(SWIFT_BENCHMARK_BUILT_STANDALONE)
    set(LLVM_HAVE_OPT_VIEWER_MODULES 1)
  endif()
else()
  set (SWIFT_BENCHMARK_OPT_VIEWER
    "${LLVM_BUILD_MAIN_SRC_DIR}/tools/opt-viewer/opt-viewer.py")
endif()

# Options for other configurations
set(BENCHOPTS_MULTITHREADED
    "-whole-module-optimization" "-num-threads" "4")
set(BENCHOPTS_SINGLEFILE "")

configure_build()

#===-----------------------------------------------------------------------===#
# SDK Configuration
#===-----------------------------------------------------------------------===#

configure_sdks()

#===---------------------------------------------------------------------===#
# Statement of Configuration for Build Users
#===---------------------------------------------------------------------===#

message("--")
message("-- Swift Benchmark Suite:")
message("--   SWIFT_BENCHMARK_BUILT_STANDALONE = ${SWIFT_BENCHMARK_BUILT_STANDALONE}")
message("--   SWIFT_EXEC = ${SWIFT_EXEC}")
message("--   SWIFT_BENCHMARK_EXTRA_FLAGS = ${SWIFT_BENCHMARK_EXTRA_FLAGS}")
message("--   SWIFT_LIBRARY_PATH = ${SWIFT_LIBRARY_PATH}")
message("--   CLANG_EXEC = ${CLANG_EXEC}")
message("--   SWIFT_OPTIMIZATION_LEVELS = ${SWIFT_OPTIMIZATION_LEVELS}")
message("--   ONLY_PLATFORMS = ${ONLY_PLATFORMS}")

message("--   found platforms: ${platforms}")
message("--   found sdks:")
foreach(sdk ${sdks})
  message("--     ${sdk}")
endforeach()

#===---------------------------------------------------------------------===#
# Build Rule Generation
#===---------------------------------------------------------------------===#

set(executable_targets)

set(srcdir "${CMAKE_CURRENT_SOURCE_DIR}")

if(SWIFT_BENCHMARK_BUILT_STANDALONE)
  set(swift-bin-dir "${CMAKE_BINARY_DIR}/bin")
else()
  get_filename_component(swift-bin-dir "${SWIFT_EXEC}" DIRECTORY)
endif()

set(benchmark-bin-dir "${CMAKE_CURRENT_BINARY_DIR}/bin")
set(benchmark-lib-dir "${CMAKE_CURRENT_BINARY_DIR}/lib")
set(benchmark-lib-swift-dir "${CMAKE_CURRENT_BINARY_DIR}/lib/swift")

file(MAKE_DIRECTORY "${swift-bin-dir}")
file(MAKE_DIRECTORY "${benchmark-bin-dir}")
file(MAKE_DIRECTORY "${benchmark-lib-dir}")
file(MAKE_DIRECTORY "${benchmark-lib-swift-dir}")

# Compile the perf test suite for each platform
foreach(platform ${platforms})
  swift_benchmark_compile(PLATFORM ${platform})
endforeach()

add_subdirectory(scripts)
