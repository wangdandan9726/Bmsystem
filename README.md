// DepartmentDlg1.cpp : implementation file
//

#include "stdafx.h"
#include "StudentInfoManage.h"
#include "DepartmentDlg.h"


#ifdef _DEBUG
#define new DEBUG_NEW
#undef THIS_FILE
static char THIS_FILE[] = __FILE__;
#endif

/////////////////////////////////////////////////////////////////////////////
// CDepartmentDlg dialog


CDepartmentDlg::CDepartmentDlg(CWnd* pParent /*=NULL*/)
	: CDialog(CDepartmentDlg::IDD, pParent)
{
	//{{AFX_DATA_INIT(CDepartmentDlg)
	m_strCode = _T("");
	m_strInfo = _T("");
	m_strName = _T("");
	//}}AFX_DATA_INIT
}


void CDepartmentDlg::DoDataExchange(CDataExchange* pDX)
{
	CDialog::DoDataExchange(pDX);
	//{{AFX_DATA_MAP(CDepartmentDlg)
	DDX_Control(pDX, IDC_LIST1, m_ctrList);
	DDX_Control(pDX, IDC_BUTTON_SAVE, m_bntSave);
	DDX_Control(pDX, IDC_BUTTON_NEW, m_bntNew);
	DDX_Control(pDX, IDC_BUTTON_MODIFY, m_bntModify);
	DDX_Control(pDX, IDC_BUTTON_DELETE, m_bntDelete);
	DDX_Text(pDX, IDC_EDIT_CODE, m_strCode);
	DDX_Text(pDX, IDC_EDIT_INFO, m_strInfo);
	DDX_Text(pDX, IDC_EDIT_NAME, m_strName);
	//}}AFX_DATA_MAP
}


BEGIN_MESSAGE_MAP(CDepartmentDlg, CDialog)
	//{{AFX_MSG_MAP(CDepartmentDlg)
	ON_BN_CLICKED(IDC_BUTTON_NEW, OnButtonNew)
	ON_BN_CLICKED(IDC_BUTTON_SAVE, OnButtonSave)
	ON_BN_CLICKED(IDC_BUTTON_MODIFY, OnButtonModify)
	ON_BN_CLICKED(IDC_BUTTON_DELETE, OnButtonDelete)
	ON_NOTIFY(NM_CLICK, IDC_LIST1, OnClickList1)
	//}}AFX_MSG_MAP
END_MESSAGE_MAP()

/////////////////////////////////////////////////////////////////////////////
// CDepartmentDlg message handlers

void CDepartmentDlg::OnButtonNew() 
{
	// TODO: Add your control notification handler code here
	m_strName	= "";
	m_strCode	= "";
	m_strInfo	= "";

	m_bntSave.EnableWindow();
	m_bntNew.EnableWindow(FALSE);
	m_bntDelete.EnableWindow(FALSE);
	m_bntModify.EnableWindow(FALSE);
	UpdateData(FALSE);		
}

void CDepartmentDlg::OnButtonSave() 
{
	// TODO: Add your control notification handler code here
	UpdateData();


	if(m_strName=="")
	{
		AfxMessageBox("请输入系名！");
		return;
	}
	if(m_strCode=="")
	{
		AfxMessageBox("请输入系别代码！");
		return;
	}

	CString strSQL;

	strSQL.Format("select * from department where code='%s'",m_strCode);
	if(!m_recordset.Open(AFX_DB_USE_DEFAULT_TYPE,strSQL))
	{
		MessageBox("打开数据库失败!","数据库错误",MB_OK);
		return ;
	}	
	if(	m_recordset.GetRecordCount()!=0)
	{
		AfxMessageBox("当前编码已经存在！请重新输入！");
		m_strCode = "";
		UpdateData(FALSE);
		m_recordset.Close();
		return;
	}
	
	m_recordset.AddNew();
	m_recordset.m_name		=	m_strName		;	
	m_recordset.m_code		=	m_strCode		;	
	m_recordset.m_brief		=	m_strInfo		;
	m_recordset.Update();

	m_recordset.Close();
	
	RefreshData();	
}

void CDepartmentDlg::OnButtonModify() 
{
	// TODO: Add your control notification handler code here
	UpdateData();
	int i = m_ctrList.GetSelectionMark();
	if(0>i)
	{
		MessageBox("请选择一条记录进行修改！");
		return;
	}

	if(m_strName=="")
	{
		AfxMessageBox("请输入系名！");
		return;
	}
	if(m_strCode=="")
	{
		AfxMessageBox("请输入系别代码！");
		return;
	}

	CString strSQL;
	strSQL.Format("select * from department where code= '%s' ",m_ctrList.GetItemText(i,0));
	if(!m_recordset.Open(AFX_DB_USE_DEFAULT_TYPE,strSQL))
	{
		MessageBox("打开数据库失败!","数据库错误",MB_OK);
		return ;
	}
	m_recordset.Edit();
	m_recordset.m_name		=	m_strName		;	
	m_recordset.m_code		=	m_strCode		;	
	m_recordset.m_brief		=	m_strInfo		;
	m_recordset.Update();

	m_recordset.Close();
	
	RefreshData();	
}

void CDepartmentDlg::OnButtonDelete() 
{
	// TODO: Add your control notification handler code here
	int i = m_ctrList.GetSelectionMark();
	if(0>i)
	{
		MessageBox("请选择一条记录进行删除！");
		return;
	}

	CString strSQL;
	strSQL.Format("select * from department where code= '%s' ",m_ctrList.GetItemText(i,0));
	if(!m_recordset.Open(AFX_DB_USE_DEFAULT_TYPE,strSQL))
	{
		MessageBox("打开数据库失败!","数据库错误",MB_OK);
		return ;
	}
	//删除该记录
	m_recordset.Delete();
	m_recordset.Close();

	//更新用户界面
	RefreshData();

	m_strName	= "";
	m_strCode	= "";
	m_strInfo	= "";
	UpdateData(FALSE);		
}

void CDepartmentDlg::OnClickList1(NMHDR* pNMHDR, LRESULT* pResult) 
{
	// TODO: Add your control notification handler code here
	CString strSQL;
	UpdateData(TRUE);
	int i = m_ctrList.GetSelectionMark();
	strSQL.Format("select * from department where code='%s'",m_ctrList.GetItemText(i,0));
	if(!m_recordset.Open(AFX_DB_USE_DEFAULT_TYPE,strSQL))
	{
		MessageBox("打开数据库失败!","数据库错误",MB_OK);
		return ;
	}	
	m_strName		=	m_recordset.m_name;
	m_strCode		=	m_recordset.m_code;
	m_strInfo		=	m_recordset.m_brief;
	
	m_recordset.Close();	

	UpdateData(FALSE);	
	m_bntSave.EnableWindow(FALSE);
	m_bntNew.EnableWindow();
	m_bntDelete.EnableWindow();
	m_bntModify.EnableWindow();	
	*pResult = 0;
}

BOOL CDepartmentDlg::OnInitDialog() 
{
	CDialog::OnInitDialog();
	
	m_ctrList.InsertColumn(0,"系别代码");
	m_ctrList.InsertColumn(1,"系名");
	m_ctrList.InsertColumn(2,"说明");

	m_ctrList.SetColumnWidth(0,60);
	m_ctrList.SetColumnWidth(1,160);
	m_ctrList.SetColumnWidth(2,240);

	m_ctrList.SetExtendedStyle(LVS_EX_FULLROWSELECT|LVS_EX_GRIDLINES);

	//设置按钮状态
	m_bntSave.EnableWindow(FALSE);
	m_bntNew.EnableWindow(FALSE);
	m_bntDelete.EnableWindow(FALSE);
	m_bntModify.EnableWindow(FALSE);

	RefreshData();
	
	return TRUE;  // return TRUE unless you set the focus to a control
	              // EXCEPTION: OCX Property Pages should return FALSE
}
void CDepartmentDlg::RefreshData()
{
	m_ctrList.DeleteAllItems();
	m_ctrList.SetRedraw(FALSE);

	UpdateData(TRUE);
	CString strSQL;
	strSQL.Format( "select * from department ");
	if(!m_recordset.Open(AFX_DB_USE_DEFAULT_TYPE,strSQL))
	{
		MessageBox("打开数据库失败!","数据库错误",MB_OK);
		return ;
	}	
	
	int i=0;
	while(!m_recordset.IsEOF())
	{
		m_ctrList.InsertItem(i,m_recordset.m_code);
		m_ctrList.SetItemText(i,1,m_recordset.m_name);
		m_ctrList.SetItemText(i,2,m_recordset.m_brief);
		i++;
		m_recordset.MoveNext();
	}
	m_recordset.Close();
	m_ctrList.SetRedraw(TRUE);
	//设置按钮状态
	m_bntSave.EnableWindow(FALSE);
	m_bntNew.EnableWindow();
	m_bntDelete.EnableWindow(FALSE);
	m_bntModify.EnableWindow(FALSE);
}

