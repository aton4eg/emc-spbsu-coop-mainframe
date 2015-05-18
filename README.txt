EMCPROJ содержит тест производительности методов трассировки под z/OS. Зовется TESTSYS. Запускать EMCPROJ.CNTL(TESTSYS). Состоит из EMCPROJ.CNTL(TESTSYS) EMCPROJ.SRC(SVCUPD1) EMCPROJ.SRC(SVC241) EMCPROJ.SRC(APPRTEST).
DACF содержит трассировщик на SVC 241 запускать DACF.CNTF(SPYMCLG).
CCWPROJ содержит программу для чтения RRZ с 3390 через CU 3990 (CCWPROJ.SRC(CCW2) + CCWPROJ.SRC(CCWR)). Запускать CCWPROJ.CNTL(CCW).
CCWPROJ содержит программу для чтения и записи на 3370 через CU 3880 (CCWPROJ.SRC(CCWF) + CCWPROJ.SRC(CCWRF)). Запускать CCWPROJ.CNTL(CCWF).
