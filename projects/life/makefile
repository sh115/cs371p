.PHONY: Life.log

.DEFAULT_GOAL := test

FILES :=         \
    .gitignore   \
    Life.h       \
    Life.log     \
    html         \
    makefile     \
    RunLife.c++  \
    RunLife.in   \
    RunLife.out  \
    TestLife.c++ \
    TestLife.out

# uncomment these:
#    .travis.yml                 \
#    Life-tests/EID-RunLife.in   \
#    Life-tests/EID-RunLife.out  \
#    Life-tests/EID-TestLife.c++ \
#    Life-tests/EID-TestLife.out \

ifeq ($(shell uname), Darwin)                                           # Apple
    CXX          := g++
    INCLUDE      := /usr/local/include
    CXXFLAGS     := -pedantic -std=c++11 -I$(INCLUDE) -Wall -Weffc++
    LIB          := /usr/local/lib
    LDFLAGS      := -lboost_serialization -lgtest_main
    CLANG-CHECK  := clang-check
    GCOV         := gcov
    GCOVFLAGS    := -fprofile-arcs -ftest-coverage
    VALGRIND     := valgrind
    DOXYGEN      := doxygen
    CLANG-FORMAT := clang-format
else ifeq ($(CI), true)                                                 # Travis CI
    CXX          := g++-5
    INCLUDE      := /usr/include
    CXXFLAGS     := -pedantic -std=c++11 -Wall -Weffc++
    LIB          := $(PWD)/gtest
    LDFLAGS      := -lboost_serialization -lgtest -lgtest_main -pthread
    CLANG-CHECK  := clang-check
    GCOV         := gcov-5
    GCOVFLAGS    := -fprofile-arcs -ftest-coverage
    VALGRIND     := valgrind
    DOXYGEN      := doxygen
    CLANG-FORMAT := clang-format
else ifeq ($(shell uname -p), unknown)                                  # Docker
    CXX          := g++
    INCLUDE      := /usr/include
    CXXFLAGS     := -pedantic -std=c++11 -Wall -Weffc++
    LIB          := /usr/lib
    LDFLAGS      := -lboost_serialization -lgtest -lgtest_main -pthread
    CLANG-CHECK  := clang-check
    GCOV         := gcov
    GCOVFLAGS    := -fprofile-arcs -ftest-coverage
    VALGRIND     := valgrind
    DOXYGEN      := doxygen
    CLANG-FORMAT := clang-format-3.5
else                                                                    # UTCS
    CXX          := g++-4.8
    INCLUDE      := /usr/include
    CXXFLAGS     := -pedantic -std=c++11 -Wall -Weffc++
    LIB          := /usr/lib
    LDFLAGS      := -lboost_serialization -lgtest -lgtest_main -pthread
    CLANG-CHECK  := clang-check-3.8
    GCOV         := gcov-4.8
    GCOVFLAGS    := -fprofile-arcs -ftest-coverage
    VALGRIND     := valgrind
    DOXYGEN      := doxygen
    CLANG-FORMAT := clang-format-3.8
endif

Life-tests:
	git clone https://github.com/cs371p-fall-2016/Life-tests.git

html: Doxyfile Life.h RunLife.c++ TestLife.c++
	doxygen Doxyfile

Life.log:
	git log > Life.log

Doxyfile:
	doxygen -g

# Make the following edits to Doxyfile.
# EXTRACT_ALL            = YES
# EXTRACT_PRIVATE        = YES
# EXTRACT_STATIC         = YES

RunLife: Life.h RunLife.c++
	$(CXX) $(CXXFLAGS) RunLife.c++ -o RunLife
	-$(CLANG-CHECK) -extra-arg=-std=c++11          RunLife.c++  --
	-$(CLANG-CHECK) -extra-arg=-std=c++11 -analyze RunLife.c++  --

RunLife.tmp: RunLife
	./RunLife < RunLife.in > RunLife.tmp
	diff RunLife.tmp RunLife.out

TestLife: Life.h TestLife.c++
	$(CXX) $(CXXFLAGS) $(GCOVFLAGS) TestLife.c++ -o TestLife $(LDFLAGS)
	-$(CLANG-CHECK) -extra-arg=-std=c++11          TestLife.c++ --
	-$(CLANG-CHECK) -extra-arg=-std=c++11 -analyze TestLife.c++ --

TestLife.tmp: TestLife
	$(VALGRIND) ./TestLife                                     >  TestLife.tmp 2>&1
	-$(GCOV) -b TestLife.c++ | grep -A 5 "File 'Life.h'"       >> TestLife.tmp
	-$(GCOV) -b TestLife.c++ | grep -A 5 "File 'TestLife.c++'" >> TestLife.tmp
	cat TestLife.tmp

check:
	@not_found=0;                                 \
    for i in $(FILES);                            \
    do                                            \
        if [ -e $$i ];                            \
        then                                      \
            echo "$$i found";                     \
        else                                      \
            echo "$$i NOT FOUND";                 \
            not_found=`expr "$$not_found" + "1"`; \
        fi                                        \
    done;                                         \
    if [ $$not_found -ne 0 ];                     \
    then                                          \
        echo "$$not_found failures";              \
        exit 1;                                   \
    fi;                                           \
    echo "success";

clean:
	rm -f  *.bin
	rm -f  *.db
	rm -f  *.gcda
	rm -f  *.gcno
	rm -f  *.gcov
	rm -f  *.plist
	rm -f  Life.log
	rm -f  Doxyfile
	rm -f  RunLife
	rm -f  RunLife.tmp
	rm -f  TestLife
	rm -f  TestLife.tmp
	rm -rf *.dSYM
	rm -rf html
	rm -rf latex

config:
	git config -l

format:
	$(CLANG-FORMAT) -i Life.h
	$(CLANG-FORMAT) -i RunLife.c++
	$(CLANG-FORMAT) -i TestLife.c++

status:
	make clean
	@echo
	git branch
	git remote -v
	git status

test: html Life.log RunLife.tmp TestLife.tmp Life-tests check

versions:
	which make
	make --version
	@echo
	which git
	git --version
	@echo
	which $(CXX)
	$(CXX) --version
	@echo
	ls -ald $(INCLUDE)/boost
	@echo
	ls -ald $(INCLUDE)/gtest
	@echo
	ls -al /usr/lib/*boost*.a
	@echo
	ls -al $(LIB)/*gtest*.a
	@echo
	which $(CLANG-CHECK)
	-$(CLANG-CHECK) --version
	@echo
	which $(GCOV)
	$(GCOV) --version
	@echo
	which $(VALGRIND)
	$(VALGRIND) --version
	@echo
	which $(DOXYGEN)
	$(DOXYGEN) --version
	@echo
	which $(CLANG-FORMAT)
	-$(CLANG-FORMAT) --version
