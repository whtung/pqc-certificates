# Makefile
#
# PQC Certificates Tests

# Name for this provider
PROVIDER_NAME := OQS

# Directories for (possibly) multiple products
# supported by the same provider
# DIRS := <product_1> <product_2> ...
DIRS := oqsprovider

# Script for Generating the artifacts (if any)
GEN_SCRIPT := ./gen_r4.sh
GEN_LOGFILE := logs/generate_log.txt

# Script for Verifying the artifacts (if any)
VERIFY_SCRIPT := ./check_r4.sh
VERIFY_LOGFILE := logs/verify_log.txt

# Time
NOW := `date +%Y%m%d`

# Default Target
.PHONY: help

# Help / Redirect Target
help:
	@echo ; \
	 echo "USAGE: make [ all, unzip, verify, cross_verify ] " ; \
	 echo && exit 1

# ====================
# Requirements Section
# ====================

requirements: requirements/unzip requirements/generate requirements/verify

# Decompress Requirements
requirements/unzip:
	@echo && echo "[ $(PROVIDER_NAME) ] Checking Requirements (Unzip):"
	@unzip=`which unzip` && [ -f "$$unzip" ] || exit 50
	@echo "[ $(PROVIDER_NAME) ] Done (Ok)"

# Generation Requirements
requirements/generate:
	@[ -d logs ] || mkdir -p "logs"
	@echo && echo "[ $(PROVIDER_NAME) ] Checking Requirements (Generate):"
	@generate=`which openssl` && [ -f "$$generate" ] || exit 100
	@echo "[ $(PROVIDER_NAME) ] Done (Ok)"

# Verification Requirements
requirements/verify:
	@[ -d logs ] || mkdir -p "logs"
	@echo && echo "[ $(PROVIDER_NAME) ] Checking Requirements (Verify):"
	@verify=`which openssl` && [ -f "$$verify" ] || exit 150
	@echo "[ $(PROVIDER_NAME) ] Done (Ok)"

# ==========================
# Functional targets Section
# ==========================

# Generic Target
all: unzip generate verify cross_verify

# Decompress Target
unzip: requirements/unzip
	@for i in $(DIRS) ; do \
	   echo && echo "[ $(PROVIDER_NAME) ] Uncompressing Artifacts for $$i:" ; \
	   if [ -f "$$i/artifacts.zip" ] ; then \
	     result=`cd "$$i" && unzip -q "artifacts.zip"` ; \
	   fi ; \
	   echo "[ $(PROVIDER_NAME) ] Done" ; \
	 done

# Generate Target, only executed if the
# `gen.sh` script is present in the provider
generate: requirements/generate
	@if [ -f "$(GEN_SCRIPT)" ] ; then \
		echo "[ $(PROVIDER_NAME) ] Generating Artifacts:" ; \
		result=`$(GEN_SCRIPT) 2>&1 > "$(GEN_LOGFILE)"` ; \
		if [ $$? -gt 0 ] ; then \
			echo "[ $(PROVIDER_NAME) ] Done (Errors)" ; \
		else \
			echo "[ $(PROVIDER_NAME) ] Done (Ok)" ; \
		fi ; \
		echo "$$result" >> "$(GEN_LOGFILE)" ; \
	 fi

# Verify Target, only executed if the
# `check.sh` script is present in the provider
verify: requirements/verify
	@if [ -f "$(VERIFY_SCRIPT)" ] ; then \
		echo && echo "[ $(PROVIDER_NAME) ] Verifying Artifacts:" ; \
		echo "----- BEGIN PROVIDER $(PROVIDER_NAME) -----" > $(VERIFY_LOGFILE); \
		echo "TEST RUN: $(NOW)" >> $(VERIFY_LOGFILE); \
		for i in $(DIRS) ; do \
			echo "[ $(PROVIDER_NAME) ] Product: $$i" ; \
			echo "PRODUCT : $$i" >> "$(VERIFY_LOGFILE)" ; \
			echo >> "$(VERIFY_LOGFILE)" ; \
			result=`$(VERIFY_SCRIPT) "$$i" 2>>"$(VERIFY_LOGFILE)" | tee -a "$(VERIFY_LOGFILE)"`; \
			echo "$$result" >> "$(VERIFY_LOGFILE)" ; \
			echo >> "$(VERIFY_LOGFILE)"; \
		done ; \
		echo "----- END PROVIDER $(PROVIDER_NAME) -----" >> $(VERIFY_LOGFILE); \
		echo "[ $(PROVIDER_NAME) ] Done (Ok)." && echo ; \
	 fi

# Cross Verify Target, it invokes the same verify
# script but it uses the base directory of a different
# provider to make use of another provider's artifacts
cross_verify: requirements/verify
	@if [ -f "$(VERIFY_SCRIPT)" ] ; then \
		echo && echo "[ $(PROVIDER_NAME) ] Cross Verifying Artifacts:" ; \
		echo > "$(VERIFY_LOGFILE)" ; \
		${VERIFY_SCRIPT} ../bc/java-artifacts 2>&1 > compatMatrices/bc_$(PROVIDER_NAME) ; \
		${VERIFY_SCRIPT} ../botan/default 2>&1 > compatMatrices/botan_$(PROVIDER_NAME) ; \
		${VERIFY_SCRIPT} ../cryptonext/default 2>&1 > compatMatrices/cryptonext_$(PROVIDER_NAME) ; \
		${VERIFY_SCRIPT} ../corey-digicert/default 2>&1 > compatMatrices/corey-digicert_$(PROVIDER_NAME); \
		${VERIFY_SCRIPT} ../entrust/default 2>&1 > compatMatrices/entrust_$(PROVIDER_NAME) ; \
		${VERIFY_SCRIPT} oqsprovider 2>&1 > compatMatrices/oqsprovider_$(PROVIDER_NAME) ; \
		echo "[ $(PROVIDER_NAME) ] Done (Ok)." && echo; \
	 fi

# Cleans up the provider's directories
clean:
	@rm -rf generate_log.txt verify_log.txt
	@for i in $(DIRS) ; do \
	   rm -rf $$i/artifacts ; \
	   rm -rf $$i/1.3.* ; \
	 done

distclean: clean
	@rm -rf logs
