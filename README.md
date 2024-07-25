## AInt-This-Spec-tacular
An OpenAI driven python app to generate architecture specification documents with minimal effort


### Target Users

Mulesoft Solutions Architects, Technical Consultants, Business Analysts


###  Example Input for a used case
Fill up the form like this:

##### Interface Name
BPA Vehicle Data Sync


##### Processing Steps

1.FLSP (one of the BPA partners) calls Azure OIDC with Client Id and Client Secret
2. FLSP gets an OAuth token back from Azure OIDC 
3. FLSP calls vr-bpa-eapi-v1’s /junk-vehicle-registration-req endpoint with vehicle’s junk-status inquiry data (JSON) (section link here) and passes the OAuth token for security pass 
4. Upon a successful request made by FLSP, vr-bpa-eapi-v1 passes the same req payload as it is to vr-bpa-papi-v1’s /junk-vehicle-registration-req endpoint. If the req is not successful (e.g. RAML validation failed) vr-bpa-eapi-v1 throws error to FLSP processed by the common-error-handling-framework 
5. vr-bpa-papi-v1 passes the same req payload vr-bpa-sapi-v1’s /junk-vehicle-registration-req endpoint 
6. vr-bpa-sapi-v1 (transforms the payload or passes it through?) calls SF initialize REST API endpoint 
7. SF makes necessary business level validations onto the payload and eventually returns a list of External systems to call for additional vehicle info if needed, along with the possible request payloads to make calls to those external systems. Currently these External systems are limited to Vintellegence, Blaze, Melissa and NMVTIS. Note: These possible request payloads are nothing but the req payloads to corresponding existing Mule SAPI/EAPIs to call those external systems 
8. The response is passed to the vr-bpa-papi-v1
9. vr-bpa-papi-v1 then orchestrates parallel calls to those external systems via existing Mule APIs 
10. Out of the 4 possible external systems vr-bpa-papi-v1 gets responses from Melissa, Vintellegence and Blaze but gets only an ack from NMVTIS (since that is an asynchronous process). NOTE: a. NMVTIS rest is synced to SF asynchronously via vr-nmvtis-sol-aamva-sapi b. In case of partial success (e.g. only 2 external systesm responded with success but another responded with failure), Mule does not stop the process, rather retain the error and moves ahead to step 10 
11. vr-bpa-papi-v1 combines the response payloads (except NMVTIS) and calls vr-bpa-sapi-v1 (endpoint name here) 
12.vr-bpa-sapi-v1 calls SF Finalize REST API endpoint (endpoint here whenever available) and passes the combined payload 
13.vr-bpa-sapi-v1 retries x times to allow SF to get the NMVTIS info fed (unless SF responds with a 200 ok success payload at the first try) 
14. After the retries SF always responds with a 200 ok either with a success payload (print data info like barcode info etc.) or an error payload (business validation errors— ICD section link here) 
15. vr-bpa-sapi-v1 passes the same 200 ok SF payload back to vr-bpa-papi-v1 
16. vr-bpa-papi-v1 passes the same 200 ok SF payload back to vr-bpa-eapi-v1 
17. vr-bpa-eapi-v1 passes the same 200 ok SF payload back to FLSP 



##### Networking

MyComp and PI are 2 networks here. 
My company named 'MyComp' is an AWS private cloud. My Organization, named 'MyOrg' is VPC which is a part of the MyComp and ranging 10.1.0.0/16. 
MyOrg has 4 subnets, 2 private subnets and 2 public subnets . Mulesoft Anypoint Platform or 'MAP' is one private subnet ranging 10.1.0.0/20. 
MAP is connected to a Public subnet names PS1 subnetted 10.1.16.0/20.  Salesforce Data Cloud, names as 'SFDC' is the second private subnet deployed in MyOrg 10.1.32.0/24. SFDC is connected to the 2nd public subnet named PS2 subnetted 10.1.48.0/20. PS1 is connected to NAT1. 
PS2 is connected to NAT2. NAT1 and NAT2 are NAT gateways. 
Both NAT1 and NAT2 both are connected to the Public Internet called PI via an Internet gateway. 
PI resides totally outside MyComp and not a part of MyComp

##### Canonical JSON

{\"junkVehicleRequest\":{\"accountNumber\":\"45CD3\",\"allocatedCounty\":\"34\",\"certNonOperationDate\":null,\"certNonOperationIndc\":null,\"certificationDate\":\"2023-03-21\",\"certificationIndc\":\"C\",\"clearingIndc\":null,\"costValue\":null,\"dealerDismantlerNumber\":23,\"equipNum\":\"5\",\"feeAcceptanceIndc\":\"N\",\"fileCode\":\"R\",\"firstPartnerId\":\"356732\",\"fuelType\":\"gas\",\"grossCombinedWeight\":\"143\",\"grossVehicleWeight\":\"456\",\"lastTransferDate\":\"2023-03-21\",\"lengthInches\":null,\"lienholderNameOnRecord\":null,\"make\":\"BMW\",\"musselFee\":null,\"numOfTransfers\":3,\"ownerAddress\":[{\"street1\":\"435 cali st\",\"street2\":\"\",\"street3\":\"\",\"city\":\"Sacramento\",\"state\":\"CA\",\"zip\":\"56782\",\"county\":25}],\"ownerInfo\":[{\"codeDlnCurr\":\"DS\",\"nameTxt\":\"Test Driver\",\"typeIndc\":\"SD\"}],\"ownerNameOnRecord\":null,\"ownershipCertIssueDate\":null,\"planNonOper\":\"Y\",\"priorPlateWithOwnerDisp\":null,\"priorUseTax\":null,\"rdfCode\":[\"1\",\"2\"],\"rdfIndicator\":\"Y\",\"regPlateNumber\":\"CAS539D\",\"repossessionDate\":null,\"secondPartnerId\":\"DGF34\",\"tnDate\":\"2023-03-21\",\"transCode\":\"N\",\"typeLicenseCode\":\"D\",\"vinHin\":\"1G6KY54951U110563\",\"vlfWgtExempt\":\"\"}}


