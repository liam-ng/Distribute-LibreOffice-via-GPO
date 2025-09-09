# Distribute-LibreOffice-via-GPO
Create LibreOffice GPO to distribute on workations. Placed Installer at sysvol. Install at startup.
Hopefully it will help others to deploy LibreOffice via GPO in the future.

# Useful Docs
- Official Document for unattended deployment[https://wiki.documentfoundation.org/Deployment_and_Migration]
- Dicussion to deploy via GPO [https://bugs.documentfoundation.org/show_bug.cgi?id=45750] (TLDR Solution: MS Orca 🐳)

# Steps
1. Download LibreOffice msi from [https://www.libreoffice.org/download/download-libreoffice/]
2. Download Windows SDK .iso from [https://developer.microsoft.com/en-us/windows/downloads/windows-sdk/]
3. Open the iso using File Explorer / Zip
4. Inside the "Installers" folders, you will find `Orca-x86_en-us.msi` [https://learn.microsoft.com/en-us/windows/win32/msi/orca-exe]
5. Install Orca
6. Right-click LibreOffice msi, you will find `Edit with Orca`
7. From top navigation menu click open: View > Summary Information...
8. Next to Languages, remove all other language codes except 1033 for English. Other languages can be found at [https://learn.microsoft.com/en-us/previous-versions/windows/internet-explorer/ie-developer/?redirectedfrom=MSDN]
9. Save as a new msi file. This is the msi file we want to add to GPO.
10. From top navigation menu click open: Transform > New Transform 
11. From the left menu, click open Property
12. Add new properties or update the current value to your liking, see the next section for details. 
13. From top navigation menu click open: Transform > Gernate Transform... it will save the property changes that you did into a .mst file. This is the tranform file we want to include when we deploy the LibreOffice.
14. Place both modified LibreOffice .msi and .mst in a accessible shared folder
15. Add permissions to folder if needed 
16. Create a new GPO at ADDC
17. Go to Computer Configuration > Policies > Software Settings > Software Instllation
18. Right click on right pane: New > Package...
19. Select the modified LibreOffice .msi from shared folder
20. **Important** Select Advanced, it may take a minute to finish
21. Double-click the LibreOffice to open the Properties window
22. Under Modifications tab, add the tramsform file .mst from shared folder
23. (Optional) Add the following 2 configurations in the same / separate GPO to ensure succuessful installation
    -	Under Computer Configuration > Policies > Administrative Templates > System
	- 	Inside `Group Policy`: `Always wait for the network at computer startup and logon` = Enabled
	-	Inside `Logon`: `Specify startup policy processing wait time. Set Amount of time to wait (in seconds)` = 30

25. Assign GPO to your test OU / workstation OU
26. Done!

## MSI Package Information
	Language: 1033	Specify only English language for installation because of Microsoft lanaguate 256 characters limiataion

## Add / Update MSI Property
- `UI_LANGS=en_US`	English for UI
- `ADDLOCAL=ALL`	Add all features & core components, specific items can be removed using `REMOVE=...`
- `REMOVE=gm_o_Onlineupdate,...`	Disable online update, "Remove" property is comma-separated
- `ISCHECKFORPRODUCTUPDATES=0`	Disable online update
- `REGISTER_ALL_MSO_TYPES=1`	Register Libre office as default applications. Can be format specific using REGISTER_XLSX=1;
		
	
- And add these mess inside REMOVE (comma-separated) if you want to remove language directories other than English to save space:
`REMOVE=gm_r_ex_Dictionary_Af,gm_r_ex_Dictionary_An,gm_r_ex_Dictionary_Ar,gm_r_ex_Dictionary_Be,gm_r_ex_Dictionary_Bg,gm_r_ex_Dictionary_Bn,gm_r_ex_Dictionary_Bo,gm_r_ex_Dictionary_Br,gm_r_ex_Dictionary_Pt_Br,gm_r_ex_Dictionary_Bs,gm_r_ex_Dictionary_Pt_Pt,gm_r_ex_Dictionary_Ca,gm_r_ex_Dictionary_Cs,gm_r_ex_Dictionary_Da,gm_r_ex_Dictionary_Nl,gm_r_ex_Dictionary_Et,gm_r_ex_Dictionary_Gd,gm_r_ex_Dictionary_Gl,gm_r_ex_Dictionary_Gu,gm_r_ex_Dictionary_He,gm_r_ex_Dictionary_Hi,gm_r_ex_Dictionary_Hu,gm_r_ex_Dictionary_Lt,gm_r_ex_Dictionary_Lv,gm_r_ex_Dictionary_Ne,gm_r_ex_Dictionary_No,gm_r_ex_Dictionary_Oc,gm_r_ex_Dictionary_Pl,gm_r_ex_Dictionary_Ro,gm_r_ex_Dictionary_Ru,gm_r_ex_Dictionary_Si,gm_r_ex_Dictionary_Sk,gm_r_ex_Dictionary_Sl,gm_r_ex_Dictionary_El,gm_r_ex_Dictionary_Es,gm_r_ex_Dictionary_Sv,gm_r_ex_Dictionary_Te,gm_r_ex_Dictionary_Th,gm_r_ex_Dictionary_Tr,gm_r_ex_Dictionary_Uk,gm_r_ex_Dictionary_Vi,gm_r_ex_Dictionary_Zu,gm_r_ex_Dictionary_Sq,gm_r_ex_Dictionary_Hr,gm_r_ex_Dictionary_De,gm_r_ex_Dictionary_Id,gm_r_ex_Dictionary_Is,gm_r_ex_Dictionary_Ko,gm_r_ex_Dictionary_Lo,gm_r_ex_Dictionary_Mn,gm_r_ex_Dictionary_Sr,gm_r_ex_Dictionary_Eo,gm_r_ex_Dictionary_It`

## Additional GPO
- 	`Always wait for the network at computer startup and logon = Enabled` I got error %%1274 due to failed DNS lookup to shared folder during installation at startup 
-	`Specify startup policy processing wait time. Set Amount of time to wait (in seconds): = 30` It takes forever to open the installer and default 120s was overkill
