---
title: "Recordset2.LockEdits Property (DAO)"
 
 
manager: soliver
ms.date: 3/9/2015
ms.audience: Developer
ms.topic: reference
  
localization_priority: Normal
ms.assetid: 77055f44-f8e9-ac64-ecc3-144ddb4a4558
description: "Sets or returns a value indicating the type of locking that is in effect while editing."
---

# Recordset2.LockEdits Property (DAO)

Sets or returns a value indicating the type of locking that is in effect while editing.
  
## Syntax

 *expression*  . **LockEdits**
  
 *expression*  A variable that represents a **Recordset2** object. 
  
## Remarks

The setting or return value indicates the type of locking, as specified in the following table.
  
|**Value**|**Description**|
|:-----|:-----|
|True  <br/> |Default. Pessimistic locking is in effect. The page containing the record you're editing is locked as soon as you call the Edit method.  <br/> |
|False  <br/> |Optimistic locking is in effect for editing. The page containing the record is not locked until the Update method is executed.  <br/> |
   
You can use the **LockEdits** property with updatable **[Recordset](recordset-object-dao.md)** objects. 
  
If a page is locked, no other user can edit records on the same page. If you set **LockEdits** to **True** and another user already has the page locked, an error occurs when you use the **Edit** method. Other users can read data from locked pages. 
  
If you set the **LockEdits** property to **False** and later use the **Update** method while another user has the page locked, an error occurs. To see the changes made to your record by another user, use the **[Move](recordset2-move-method-dao.md)** method with 0 as the argument; however, if you do this, you will lose your changes. 
  
When working with Microsoft Access database engine-connected ODBC data sources, the **LockEdits** property is always set to **False**, or optimistic locking. The Microsoft Access database engine has no control over the locking mechanisms used in external database servers. 
  
> [!NOTE]
> You can preset the value of **LockEdits** when you first open the **Recordset** by setting the  _lockedits_ argument of the **[OpenRecordset](connection-openrecordset-method-dao.md)** method. Setting the  _lockedits_ argument to **dbPessimistic** will set the **LockEdits** property to **True**, and setting  _lockedits_ to any other value will set the **LockEdits** property to **False**. 
  
## Example

This example demonstrates pessimistic locking by setting the **LockEdits** property to **True**, and then demonstrates optimistic locking by setting the **LockEdits** property to False. It also demonstrates what kind of error handling is required in a multiuser database environment in order to modify a field. The PessimisticLock and OptimisticLock functions are required for this procedure to run. 
  
```
Sub LockEditsX() 
 
 Dim dbsNorthwind As Database 
 Dim rstCustomers As Recordset2 
 Dim strOldName As String 
 
 Set dbsNorthwind = OpenDatabase("Northwind.mdb") 
 Set rstCustomers = _ 
 dbsNorthwind.OpenRecordset("Customers", _ 
 dbOpenDynaset) 
 
 With rstCustomers 
 ' Store original data. 
 strOldName = !CompanyName 
 
 If MsgBox("Pessimistic locking demonstration...", _ 
 vbOKCancel) = vbOK Then 
 
 ' Attempt to modify data with pessimistic locking 
 ' in effect. 
 If PessimisticLock(rstCustomers, !CompanyName, _ 
 "Acme Foods") Then 
 MsgBox "Record successfully edited." 
 
 ' Restore original data... 
 .Edit 
 !CompanyName = strOldName 
 .Update 
 End If 
 
 End If 
 
 If MsgBox("Optimistic locking demonstration...", _ 
 vbOKCancel) = vbOK Then 
 
 ' Attempt to modify data with optimistic locking 
 ' in effect. 
 If OptimisticLock(rstCustomers, !CompanyName, _ 
 "Acme Foods") Then 
 MsgBox "Record successfully edited." 
 
 ' Restore original data... 
 .Edit 
 !CompanyName = strOldName 
 .Update 
 End If 
 
 End If 
 
 .Close 
 End With 
 
 dbsNorthwind.Close 
 
End Sub 
 
Function PessimisticLock(rstTemp As Recordset2, _ 
 fldTemp As Field, strNew As String) As Boolean 
 
 dim ErrLoop as Error 
 
 PessimisticLock = True 
 
 With rstTemp 
 .LockEdits = True 
 
 ' When you set LockEdits to True, you trap for errors 
 ' when you call the Edit method. 
 On Error GoTo Err_Lock 
 .Edit 
 On Error GoTo 0 
 
 ' If the Edit is still in progress, then no errors 
 ' were triggered; you may modify the data. 
 If .EditMode = dbEditInProgress Then 
 fldTemp = strNew 
 .Update 
 .Bookmark = .LastModified 
 Else 
 ' Retrieve current record to see changes made by 
 ' other user. 
 .Move 0 
 End If 
 
 End With 
 
 Exit Function 
 
Err_Lock: 
 
 If DBEngine.Errors.Count > 0 Then 
 ' Enumerate the Errors collection. 
 For Each errLoop In DBEngine.Errors 
 MsgBox "Error number: " &amp; errLoop.Number &amp; _ 
 vbCr &amp; errLoop.Description 
 Next errLoop 
 PessimisticLock = False 
 End If 
 
 Resume Next 
 
End Function 
 
Function OptimisticLock(rstTemp As Recordset, _ 
 fldTemp As Field, strNew As String) As Boolean 
 
 dim ErrLoop as Error 
 
 OptimisticLock = True 
 
 With rstTemp 
 .LockEdits = False 
 .Edit 
 fldTemp = strNew 
 
 ' When you set LockEdits to False, you trap for errors 
 ' when you call the Update method. 
 On Error GoTo Err_Lock 
 .Update 
 On Error GoTo 0 
 
 ' If there is no Edit in progress, then no errors were 
 ' triggered; you may modify the data. 
 If .EditMode = dbEditNone Then 
 ' Move current record pointer to the most recently 
 ' modified record. 
 .Bookmark = .LastModified 
 Else 
 .CancelUpdate 
 ' Retrieve current record to see changes made by 
 ' other user. 
 .Move 0 
 End If 
 
 End With 
 
 Exit Function 
 
Err_Lock: 
 
 If DBEngine.Errors.Count > 0 Then 
 ' Enumerate the Errors collection. 
 For Each errLoop In DBEngine.Errors 
 MsgBox "Error number: " &amp; errLoop.Number &amp; _ 
 vbCr &amp; errLoop.Description 
 Next errLoop 
 OptimisticLock = False 
 End If 
 
 Resume Next 
 
End Function 

```

