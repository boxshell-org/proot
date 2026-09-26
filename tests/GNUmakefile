DIR    = $(dir $(abspath $(firstword $(MAKEFILE_LIST))))
ROOTFS = $(DIR)/rootfs
PROOT  = $(DIR)/../src/proot
CC     = gcc
PROOT_RAW = $(PROOT)

CHECK_TESTS = $(patsubst %,check-%, $(wildcard test-*.sh) $(wildcard test-*.c))

# Tests with known failures.  They still run, but a failure is reported
# as "xfail" instead of failing the suite; an unexpected pass is
# reported as "xpass" (informational only -- several tests are
# host-dependent).  Remove entries when the underlying bug is fixed.
#
#   test-0cf405b0  execlp("/proc/self/exe") loops forever under ptrace
#                  https://github.com/boxshell-org/proot/issues/1
#   test-691786c8  execve of a script with an overlong shebang -> ELOOP
#                  https://github.com/boxshell-org/proot/issues/2
XFAIL_TESTS = test-0cf405b0 test-691786c8

# Per-test timeout in seconds so a hung test (see issue #1) is bounded
# instead of stalling the suite forever.  Exit status of a killed test
# is 124, reported below.
TEST_TIMEOUT ?= 120

.PHONY: check clean_failure check_failure setup check-%

check: | clean_failure check_failure

memcheck: PROOT_RAW := $(PROOT)
memcheck: PROOT := $(shell which valgrind) -q --error-exitcode=1 $(PROOT)
memcheck: check

clean_failure:
	@rm -f failure

check_failure: $(CHECK_TESTS)
	@bash -c '! test -e failure'

check-%.sh: %.sh setup
	$(Q)env ROOT_RAW="$(PROOT_RAW)" PROOT="$(PROOT)" ROOTFS=$(ROOTFS) timeout -k 5 $(TEST_TIMEOUT) sh -ex $< $(silently); $(call check,$*)

check-%.c: $(ROOTFS)/bin/% setup
	$(call check_c,$*,$(PROOT) -b /proc -r $(ROOTFS) /bin/$*)

# Special cases.
check-test-bdc90417.c: test-bdc90417
	$(call check_c,$<,$(PROOT) -w . ./$<)

# Not supported anymore.
# check-test-af062114.c: test-af062114
# 	$(call check_c,$<,$(PROOT) -v -1 -q /bin/true -b . -b /lib -b /lib64 -b /proc -r $(ROOTFS) ./$< | grep -- --inhibit-rpath)
# 	$(call check_c,$<,$(PROOT) -v -1 -q /bin/true / ./$< | grep '^./$<') deprecated

check-test-5bed7141.c: test-5bed7141
	$(call check_c,$<,$(PROOT) ./$<)

check-test-5bed7143.c: test-5bed7143
	$(call check_c,$<,$(PROOT) -r $(ROOTFS) -b . ./$<)
	$(call check_c,$<,$(PROOT) ./$<)

check-test-16573e73.c: test-16573e73
	$(call check_c,$<,$(PROOT) ./$<)
	$(call check_c,$<,$(PROOT) ./$< 1)

check-test-82ba4ba1.c: test-82ba4ba1
	$(call check_c,$<,$(PROOT) -0 ./$<)
	$(call check_c,$<,! $(PROOT) -i 123:456 ./$<)

check-test-kkkkkkkk.c: test-kkkkkkkk
	$(call check_c,$<,$(PROOT) ./$<)

check-test-25069c12.c: test-25069c12
	$(call check_c,$<,$(PROOT) ./$<)

check-test-25069c13.c: test-25069c13
	$(call check_c,$<,$(PROOT) ./$<)

check-test-1ffc8309.c: test-1ffc8309
	$(call check_c,$<,env PROOT_FORCE_KOMPAT=1 $(PROOT) -k $(shell uname -r) ./$<)

check-test-c5a7a0f0.c: test-c5a7a0f0
	$(call check_c,$<,$(PROOT) -0 ./$<)
	$(call check_c,$<,! $(PROOT) -i 123:456 ./$<)

check-test-a3e68988.c: test-a3e68988
	@which gdb >/dev/null 2>&1 || rm -f $<
	$(call check_c,$<,$(PROOT) gdb -return-child-result -ex run -ex quit ./$<)

check-test-c47aeb7d.c: test-c47aeb7d
	@which gdb >/dev/null 2>&1 || rm -f $<
	$(call check_c,$<,$(PROOT) gdb -return-child-result -ex run -ex quit ./$<)

check-test-fdf487a0.c: test-fdf487a0
	$(call check_c,$<,echo test | $(PROOT) ./$<)

check-test-iiiiiiii.c: test-iiiiiiii
	$(call check_c,$<,echo test | env PROOT_DONT_POLLUTE_ROOTFS=1 $(PROOT) -b /bin:/this_shall_not_exist_outside_proot ./$<)

check-test-9c07fad8.c: test-9c07fad8
	$(call check_c,$<,$(PROOT) ./$<)

check-test-fa205b56.c: test-fa205b56
	$(call check_c,$<,$(PROOT) ./$<)

check_c = $(Q)if [ -e $< ]; then			\
		timeout -k 5 $(TEST_TIMEOUT) $(2) $(silently); $(call check,$(1))	\
	else						\
		echo "  CHECK	$(1) skipped";		\
	fi

# $(1) is the test name.  Members of XFAIL_TESTS fail softly.
check = case "$$?" in					\
		0)   echo "  CHECK	$(1) $(if $(filter $(1),$(XFAIL_TESTS)),xpass,ok)";; \
		124) $(if $(filter $(1),$(XFAIL_TESTS)),echo "  CHECK	$(1) xfail (timeout)",echo "  CHECK	$(1) FAILED (timeout)"; touch failure);; \
		125) echo "  CHECK	$(1) skipped";;	\
		*)   $(if $(filter $(1),$(XFAIL_TESTS)),echo "  CHECK	$(1) xfail",echo "  CHECK	$(1) FAILED"; touch failure);; \
	esac

######################################################################
# Build a clean rootfs

ROOTFS_BIN = $(ROOTFS)/bin/true $(ROOTFS)/bin/false    				 \
       $(ROOTFS)/bin/pwd  $(ROOTFS)/bin/readlink $(ROOTFS)/bin/symlink		 \
       $(ROOTFS)/bin/abs-true $(ROOTFS)/bin/rel-true $(ROOTFS)/bin/echo		 \
       $(ROOTFS)/bin/argv0 $(ROOTFS)/bin/readdir $(ROOTFS)/bin/cat		 \
       $(ROOTFS)/bin/chdir_getcwd $(ROOTFS)/bin/fchdir_getcwd $(ROOTFS)/bin/argv \
       $(ROOTFS)/bin/fork-wait $(ROOTFS)/bin/ptrace $(ROOTFS)/bin/ptrace-2	 \
       $(ROOTFS)/bin/puts_proc_self_exe $(ROOTFS)/bin/exec $(ROOTFS)/bin/exec-m32 \
       $(ROOTFS)/bin/exec-suid $(ROOTFS)/bin/exec-sgid $(ROOTFS)/bin/exec-m32-suid \
       $(ROOTFS)/bin/exec-m32-sgid $(ROOTFS)/bin/getresuid $(ROOTFS)/bin/getresgid \
       $(ROOTFS)/bin/open_nofollow

ROOTFS_DIR = $(ROOTFS)/bin $(ROOTFS)/tmp

$(ROOTFS_BIN): | $(ROOTFS_DIR)

$(ROOTFS_DIR):
	@mkdir -p $@

setup: $(ROOTFS_BIN)

$(ROOTFS)/bin/abs-true:
	$(Q)ln -fs /bin/true $@

$(ROOTFS)/bin/rel-true:
	$(Q)ln -fs ./true $@

$(ROOTFS)/bin/exec-m32: exec.c
	$(Q)$(CC) -m32 -static $^ -o $@ $(silently) || true

$(ROOTFS)/bin/exec-suid: $(ROOTFS)/bin/exec
	$(Q)cp $^ $@
	$(Q)chmod u+s $@

$(ROOTFS)/bin/exec-sgid: $(ROOTFS)/bin/exec
	$(Q)cp $^ $@
	$(Q)chmod g+s $@

$(ROOTFS)/bin/exec-m32-suid: $(ROOTFS)/bin/exec-m32
	$(Q)cp $^     $@ $(silently) || true
	$(Q)chmod u+s $@ $(silently) || true

$(ROOTFS)/bin/exec-m32-sgid: $(ROOTFS)/bin/exec-m32
	$(Q)cp $^ $@      $(silently) || true
	$(Q)chmod g+s $@  $(silently) || true

.SECONDARY: $(patsubst %.c,$(ROOTFS)/bin/%, $(wildcard test-*.c))
$(ROOTFS)/bin/%: %.c
	$(Q)$(CC) -static $*.c -o $@ $(silently) || true

# Special cases.
test-bdc90417: test-bdc90417.c
	$(Q)$(CC) $< -o $@ $(silently) || true

# Not supported anymore.
# test-af062114: test-af062114.c
# 	$(Q)$(CC) $< -Wl,-rpath=foo -o $@ $(silently) || true

test-5bed7141: test-5bed7141.c
	$(Q)$(CC) $< -pthread -static -o $@ $(silently) || true

test-16573e73: test-16573e73.c
	$(Q)$(CC) $< -static -o $@ $(silently) || true

test-82ba4ba1: test-82ba4ba1.c
	$(Q)$(CC) $< -o $@ $(silently) || true

test-kkkkkkkk: test-kkkkkkkk.c
	$(Q)$(CC) $< -o $@ $(silently) || true

test-25069c12: test-25069c12.c
	$(Q)$(CC) $< -o $@ $(silently) || true

test-25069c13: test-25069c13.c
	$(Q)$(CC) $< -o $@ $(silently) || true

test-5bed7143: test-5bed7143.c
	$(Q)$(CC) $< -static -o $@ $(silently) || true

test-1ffc8309: test-1ffc8309.c
	$(Q)$(CC) $< -o $@ $(silently) || true

test-c5a7a0f0: test-c5a7a0f0.c
	$(Q)$(CC) $< -pthread -static -o $@ $(silently) || true

test-a3e68988: test-a3e68988.c
	$(Q)$(CC) $< -o $@ $(silently) || true

test-fdf487a0: test-fdf487a0.c
	$(Q)$(CC) $< -o $@ $(silently) || true

test-iiiiiiii: test-iiiiiiii.c
	$(Q)$(CC) $< -o $@ $(silently) || true

test-9c07fad8: test-9c07fad8.c
	$(Q)$(CC) -fPIE -pie $< -o $@ $(silently) || true

test-fa205b56: test-fa205b56.c
	$(Q)$(CC) $< -pthread -o $@ $(silently) || true

test-c47aeb7d: test-c47aeb7d.c
	$(Q)$(CC) $< -pthread -o $@ $(silently) || true

######################################################################
# Beautified output

V = 0
ifeq ($(V), 0)
    quiet = quiet_
    Q     = @
    silently = >/dev/null 2>&1
else
    quiet = 
    Q     = 
    silently = 
endif
