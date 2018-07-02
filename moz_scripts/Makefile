# Makefile to collect data

.DEFAULT_GOAL := help

CACHE_DIR := .cache

SERVICE_ORGS := \
	andyet \
	bugzilla \
	mozilla \
	mozilla-bteam \
	mozilla-conduit \
	mozilla-mobile \
	mozilla-releng \
	mozilla-services \
	taskcluster \

OTHER_ORGS := \
	mozilla-b2g \
	mozilla-frontend-infra \
	mozilla-neutrino \
	mozilla-partners \
	mozilla-platform-ops \

SERVICE_DBS := $(SERVICE_ORGS:=.db.json)
OTHER_DBS := $(OTHER_ORGS:=.db.json)

ALL_ORGS := $(SERVICE_ORGS) $(OTHER_ORGS)
ALL_DBS := $(SERVICE_DBS) $(OTHER_DBS)

# Local Static Rules
.PHONY: $(ALL_ORGS)
$(ALL_DBS) : %.db.json: %
	./get_branch_protections.py $@

help:
	@echo "Makefile to run repo status reports"
	@echo "run from parent directory as:"
	@echo "  make -f moz_scripts/Makefile"
	@echo ""
	@echo "Targets in this Makefile"
	@echo "  clean       remove data from prior runs"
	@echo "  get         obtain all data for service orgs"
	@echo "  report      build the per-service reports"
	@echo "  consolidate gather all per-service reports for spreadsheet import"
	@echo "              into consolidate.csv"
	@echo "  store       store latest data into foxsec-results"
	@echo "              (manual push required)"
	@echo "  open_protected_issues  open GitHub issues on repositories which"
	@echo "                         do not have branch protection enabled"
	@echo ""
	@echo "  get_others  obtain all data for non-service orgs"
	@echo "  get_all     obtain all data for all configured orgs"

list:
	@echo $(ALL_ORGS)
	@echo $(SERVICE_DBS)

clean:
	@echo rm -rf $(CACHE_DIR)
	@echo mkdir $(CACHE_DIR)

get: $(SERVICE_DBS)
get_others: $(OTHER_DBS)
get_all: $(ALL_DBS)

report:
	moz_scripts/report-by-service

consolidated.csv \
consolidate:
	@# force the sort order we want
	env LANG=en_US.UTF-8 bash -c 'cat $$(ls -1td /tmp/$$USER-report-by-service-* | head -1)/*.csv' >consolidated.csv

open_protected_issues: consolidated.csv
	moz_scripts/open_issues.py $$(grep '/' consolidated.csv | cut -d, -f1,2 | grep False$$ | cut -d, -f1)

store:
	bash -xc ' \
		tmp_dir=$$(mktemp -d /tmp/$${USER}-GitHub-Audit-XXXXXX) ; \
		echo Using $$tmp_dir for work ; \
		pushd $$tmp_dir &>/dev/null ; \
		git clone --depth 1 git+ssh://github.com/mozilla-services/foxsec-results ; \
		cd foxsec-results/github-configs ; \
		for f in ~1/*.db.json ; do \
		  jq . < $$f > $${f##*/} ; \
		done ; \
		git commit -am "$$(date +%F) github-configs update" \
		'
	@echo "Manually push from $$(ls -1td /tmp/$${USER}-GitHub-Audit-* | head -1)"
	@# force failure to ensure push done prior to anything else
	false

.PHONY: list clean get report consolidate help store