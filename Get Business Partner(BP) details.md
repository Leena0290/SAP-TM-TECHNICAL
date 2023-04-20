<a name="top"></a>
# Get Business Partner(BP)  details in SAP TM

In SAP TM, one of the common requirements is to fetch BP details like BP name from SCAC, BP  complete addresses, contact details, etc. Below are the various methods to get BP details with the help of standard methods available.

# Different methods to get BP details
 - [Retrieve By Association](#retrieve-by-association)
 - [Helper Classes](#helper-classes)
 - [Business Object](#business-object)

## Retrieve By Association
Using the business object instance, BP details can be fetched as below:
````abap
DATA: bp_key      TYPE /bobf/t_frw_key,
      bp_addr_key TYPE /bobf/t_frw_key,
      bp_address  TYPE /bofu/t_addr_addressi_k.
      
"Create instances for business objects
DATA(lo_bp_srvmgr) = /bobf/cl_tra_serv_mgr_factory=>get_service_manager( /scmtms/if_bp_c=>sc_bo_key ).

" Using Bupa instance
CALL METHOD lo_bp_srvmgr->retrieve_by_association
  EXPORTING
    iv_node_key    = /bofu/if_bupa_constants=>sc_node-root
    it_key         = bp_key
    iv_association = /bofu/if_bupa_constants=>sc_association-root-addressinformation
  IMPORTING
    et_target_key  = bp_addr_key.

CALL METHOD lo_bp_srvmgr->retrieve_by_association
  EXPORTING
    iv_node_key    = /bofu/if_bupa_constants=>sc_node-addressinformation
    it_key         = bp_addr_key
    iv_association = /bofu/if_bupa_constants=>sc_association-addressinformation-address
    iv_fill_data   = abap_true
  IMPORTING
    et_data        = bp_address.
  
IF bp_address IS NOT INITIAL.
  DATA(postal) = bp_address[ 1 ]-postal_address_id .
  SELECT addrnumber,name1,city1 FROM adrc INTO TABLE @DATA(address) WHERE addrnumber = @postal.
ENDIF.

````
> **💡 Note**<br>
> In case we do not have a direct BP key available, we can convert the business partner using below code use the same 
````abap
" Convert Alternate Key fo Buisness partner - carrier
CALL METHOD lo_bp_srvmgr->convert_altern_key
  EXPORTING
    iv_node_key   = /bofu/if_bupa_constants=>sc_node-root    " Node
    iv_altkey_key = /bofu/if_bupa_constants=>sc_alternative_key-root-partner    " Alternative Key
    it_key        =  pty_id "business partner ID
  IMPORTING
    et_key        = bp_key.
````
<p align="right">(<a href="#top">back to top</a>)</p>

## Helper Classes
With the below standard helper classes available in the system, the business partner address(postal code, organization, email, etc.) and other related details can be fetched according to the requirement.

A. The below helper class gives the business partner's address. Also, we need to pass mandatory parameters to get specific details with respect to BP like below:
````abap
"Helper Class to get Address Data
/scmtms/cl_addr_helper=>read_bupa_addr(
  EXPORTING
    it_bupa_key       = bp_key
    is_addr_read_ctrl = VALUE #( ad_postal = abap_true ad_orgname = abap_true ad_email = abap_true )
  IMPORTING
    et_bupa_addr      = DATA(lt_bp_addr) ).
````
B. This helper method can be used to fetch BP details like address, partner description, bp organizational data, etc., related to /SCMTMS/BUPA business object
 ````abap
"Helper methods for business object /SCMTMS/BUPA
data(tsp_id) = '1002612606'	.
CALL METHOD /scmtms/cl_bupa_helper=>get_bp_address
  EXPORTING
    iv_partner      = tsp_id              " CARRIER/ BP
  IMPORTING
    et_partner_addr = DATA(bp_addr).      "ADDRESS
 ````
 C. The below methods can be used with or without UI case; in UI case, pass FBI Controller for a specific business object to parameter io_controller   
 ````abap
" Read Business Partner details
/scmtms/cl_helper_ui=>read_bupa_det(
  EXPORTING
*  io_controller = io_controller
    it_bupa_key  = bp_key
    iv_bupa_det  = abap_true
    iv_addr_det  = abap_true
  IMPORTING
    et_bupa_data = DATA(lt_bupa) ).

" Read formatted address for Locations and Partners
/scmtms/cl_helper_ui=>read_addr_simple(
  EXPORTING
    it_bupa_key  = bp_key           " Key Table
*   it_bupa_adnr =                  " Link between BP and addresses (e.g. document specific)
    iv_printform =    'X'           " Printform shall be read
  RECEIVING
    rt_address   = data(lt_data)).
````
D. Using the method below, fetch the active business partner from the SCAC that is valid to the system date.
````abap
DATA :scac_code  TYPE STANDARD TABLE OF /scmtms/scacd.

APPEND  'ABCD' TO scac_code.   " <= 4
/scmtms/cl_common_bo_helper=>get_bp_from_scac
  EXPORTING
    it_scac          = scac_code
    iv_ge_valid_from = sy-datum                " If NOT 0: Don't show results which SCAC Valid From is higher
    iv_le_valid_to   = sy-datum                " If NOT 0: Don't show results which SCAC Valid To is lower
    iv_del_multiple  = abap_true               " Don't show those results if a SCAC has more than 1 BP
  IMPORTING
    et_bp_from_scac  = DATA(bp_number).
````
<p align="right">(<a href="#top">back to top</a>)</p>

## Business Object
Database tables/ CDS views can also be used to get Business Partner details as part of optimization and can be found under BO node /SCMTMS/BUPA.
To check  specific CDS view/ DB table, navigate using the below path:

Tcode - BOBX -> open the BO - /SCMTMS/BUPA -> go to Node Structure -> check  specific Data Accesses related to BP

Ex: GET SCAC code details as mentioned below:

<img src="https://user-images.githubusercontent.com/87542870/233247073-fa099d5a-ecd3-4121-aa66-d1d99a13a9cf.png" width="300" height="200">
<p align="right">(<a href="#top">back to top</a>)</p>

All these above methods can be used to get BP data as per the business requirements to get an optimized solution.
<p align="right">(<a href="#top">back to top</a>)</p>

