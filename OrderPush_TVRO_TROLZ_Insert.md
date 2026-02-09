# TVRO/TROLZ Block Insert – Z_SCM_GET_ORDER_DETAILS

## Purpose

Insert the TVRO/TROLZ route and zone population block into the **IF sy-subrc = 0** branch of the FM **Z_SCM_GET_ORDER_DETAILS**, so that when parameter `ZSCM_ORDER_SLOC_OPUSH` is found, route description, distz, medst, traztd, azone, and lzone are also filled from TVRO and TROLZ (in addition to the existing T001L/KNA1 logic).

## Source

- **Program / FM:** Z_SCM_GET_ORDER_DETAILS  
- **Source file:** `OrderPush Tcode Program Details.txt`  
- **Insert after:** Line **1557** (the `ENDIF.` that closes the “receiving_plant” / KNA11 block)  
- **Insert before:** The `ELSE.` at line 1559  

## Code to Insert

Add the following block **after line 1557** and **before the ELSE**:

```abap
        CLEAR: lw_tvro.
        READ TABLE lt_tvro INTO lw_tvro WITH KEY  route = lw_ex-route BINARY SEARCH.
        IF sy-subrc EQ 0.
          lw_item_det-route_description = lw_tvro-routid.
          lw_header_det-distz = lw_tvro-distz.
          lw_header_det-medst = lw_tvro-medst.
          IF lw_tvro-traztd IS NOT INITIAL.
            lw_header_det-traztd = lw_tvro-traztd.
          ENDIF.
        ENDIF.

        CLEAR: lw_trolz.
        READ TABLE lt_trolz INTO lw_trolz WITH KEY route = lw_ex-route BINARY SEARCH.
        IF sy-subrc = 0.
          lw_header_det-azone = lw_trolz-azone.
          lw_header_det-lzone = lw_trolz-lzone.
        ENDIF.
```

## Context (for reference)

**Before insertion** (lines 1551–1559 in source):

```abap
        IF sy-subrc = 0.
          CLEAR lw_kna11.
          READ TABLE lt_kna11 INTO lw_kna11 WITH KEY kunnr = lw_too1l-kunnr.
          IF sy-subrc = 0.
            lw_header_det-lzone = lw_kna11-lzone.
          ENDIF.
        ENDIF.
      ELSE.
```

**After insertion**, the structure should be:

```abap
        IF sy-subrc = 0.
          CLEAR lw_kna11.
          READ TABLE lt_kna11 INTO lw_kna11 WITH KEY kunnr = lw_too1l-kunnr.
          IF sy-subrc = 0.
            lw_header_det-lzone = lw_kna11-lzone.
          ENDIF.
        ENDIF.
        CLEAR: lw_tvro.
        READ TABLE lt_tvro INTO lw_tvro WITH KEY  route = lw_ex-route BINARY SEARCH.
        ... (full TVRO/TROLZ block as above)
        ENDIF.
      ELSE.
```

## Summary

| Item        | Detail                                                |
|------------|--------------------------------------------------------|
| **Object** | Function module Z_SCM_GET_ORDER_DETAILS                |
| **File**   | OrderPush Tcode Program Details.txt                   |
| **After**  | Line 1557 (`ENDIF.` after `lw_header_det-lzone = lw_kna11-lzone.`) |
| **Before** | Line 1559 (`ELSE.`)                                   |
| **Action** | Paste the TVRO/TROLZ block in the ABAP source at the position above. |
