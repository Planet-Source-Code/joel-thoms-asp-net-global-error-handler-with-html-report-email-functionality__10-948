<div align="center">

## ASP\.NET Global Error Handler \(with html report & email functionality\)


</div>

### Description

Globally capture errors and exceptions in your ASP.NET site. Admin(s) can be emailed a detailed HTML report of the error, then redirect the user to a friendly error page. 100% C#, no external objects needed.
 
### More Info
 


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[Joel Thoms](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/joel-thoms.md)
**Level**          |Beginner
**User Rating**    |5.0 (25 globes from 5 users)
**Compatibility**  |C\#, ASP\.NET
**Category**       |[Debugging and Error Handling](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/debugging-and-error-handling__10-6.md)
**World**          |[\.Net \(C\#, VB\.net\)](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/net-c-vb-net.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/joel-thoms-asp-net-global-error-handler-with-html-report-email-functionality__10-948/archive/master.zip)

### API Declarations

Copyright 2003 Joel Thoms


### Source Code

/* Author: Joel Thoms
 * Website: http://www.joel.net
 * Email: (contact me through website)
 * Date: 02.05.2003
 *
 * Copyright 2003 Joel Thoms
 *
 * Description:
 *	HtmlError generates an HTML error message from the generated Exception. HtmlError also includes
 *	a routine to email the error message to the admin(s).
 *
 *	This object can be used to capture individual errors, though it's best use is to globally capture
 *	errors using Global.asax. Both examples are provided.
 *
 *
 * Usage and Examples:
 *
 *	Here is an example on how to capture a simple division by zero error.
 *
 *	[C#]
 *		// Division by zero error example
 *		try {
 *			int x = 0;
 *			x = 1 / x;
 *		} catch (Exception Ex) {
 *			// Display Error Message to the browser
 *			Response.Write(HtmlError.getHtmlError(Ex));
 *
 *			// Don't Specify SMTP Server
 *			HtmlError.sendHtmlError(Ex, "YOUR-EMAIL@ADDRESS.COM");
 *
 *			// Specify SMTP Server
 *			//HtmlError.sendHtmlError(Ex, "YOUR-EMAIL@ADDRESS.COM", "your.smtp-server.com");
 *		}
 *
 *
 * 	[VB.NET]
 *
 *		' Division by zero error example
 *		Try
 *			Dim x As Integer = 0
 *			x = 1 / x
 *		Catch Ex As Exception
 *			' Display Error Message to the browser
 *			Response.Write(HtmlError.getHtmlError(Ex))
 *
 *			' Don't Specify SMTP Server
 *			HtmlError.sendHtmlError(Ex, "YOUR-EMAIL@ADDRESS.COM")
 *
 *			' Specify SMTP Server
 *			'HtmlError.sendHtmlError(Ex, "YOUR-EMAIL@ADDRESS.COM", "your.smtp-server.com")
 *		End Try
 *
 *
 *	Here is an example on globally capturing errors using the Global.asax file.
 *
 *	[C#]
 *
 * 		protected void Application_Error(Object sender, EventArgs e) {
 *			// Don't Specify SMTP Server
 *			HtmlError.sendHtmlError(Context.Error.GetBaseException(), "YOUR-EMAIL@ADDRESS.COM");
 *
 *			// Specify SMTP Server
 *			//HtmlError.sendHtmlError(Context.Error.GetBaseException(), "YOUR-EMAIL@ADDRESS.COM", "your.smtp-server.com");
 *
 *			// Redirect User to Friendly Error Page
 *			Response.Redirect("/error.aspx");
 *		}
 *
 *	[VB.NET]
 *		Sub Application_Error(ByVal sender As Object, ByVal e As EventArgs)
 *			' Don't Specify SMTP Server
 *			HtmlError.sendHtmlError(Context.Error.GetBaseException(), "YOUR-EMAIL@ADDRESS.COM")
 *
 *			' Specify SMTP Server
 *			'HtmlError.sendHtmlError(Context.Error.GetBaseException(), "YOUR-EMAIL@ADDRESS.COM", "your.smtp-server.com")
 *
 *			' Redirect User to Friendly Error Page
 *			Response.Redirect("/error.aspx")
 *		End Sub
 *
 *
*/
using System;
using System.Data;
using System.Web;
using System.Web.Mail;
using System.Collections.Specialized;
/// <summary>HtmlError Object.</summary>
public class HtmlError {
	public HtmlError() { }
	static public void sendHtmlError(Exception Ex, string email_address) { sendHtmlError(Ex, email_address, ""); }
	static public void sendHtmlError(Exception Ex, string email_address, string smtp_server) {
		MailMessage mail = new MailMessage();
		mail.From = "server-errors@discountasp.net";
		mail.To = email_address;
		mail.Subject = "Uncaptured Error";
		mail.Body = getHtmlError(Ex);
		mail.BodyFormat = MailFormat.Html;
		if (smtp_server.Length > 0) SmtpMail.SmtpServer = smtp_server;
		SmtpMail.Send(mail);
	}
	/// <summary>Returns HTML an formatted error message.</summary>
	static public string getHtmlError(Exception Ex) {
		// Heading Template
		const string heading = "<TABLE BORDER=\"0\" WIDTH=\"100%\" CELLPADDING=\"1\" CELLSPACING=\"0\"><TR><TD bgcolor=\"black\" COLSPAN=\"2\"><FONT face=\"Arial\" color=\"white\"><B>&nbsp;<!--HEADER--></B></FONT></TD></TR></TABLE>";
		// Error Message Header
		string html = "<FONT face=\"Arial\" size=\"5\" color=\"red\">Error - " + Ex.Message + "</FONT><BR><BR>";
		// Populate Error Information Collection
		NameValueCollection error_info = new NameValueCollection();
		error_info.Add("Message", cleanHTML(Ex.Message));
		error_info.Add("Source", cleanHTML(Ex.Source));
		error_info.Add("TargetSite", cleanHTML(Ex.TargetSite.ToString()));
		error_info.Add("StackTrace", cleanHTML(Ex.StackTrace));
		// Error Information
		html += heading.Replace("<!--HEADER-->", "Error Information");
		html += CollectionToHtmlTable(error_info);
		// QueryString Collection
		html += "<BR><BR>" + heading.Replace("<!--HEADER-->", "QueryString Collection");
		html += CollectionToHtmlTable(HttpContext.Current.Request.QueryString);
		// Form Collection
		html += "<BR><BR>" + heading.Replace("<!--HEADER-->", "Form Collection");
		html += CollectionToHtmlTable(HttpContext.Current.Request.Form);
		// Cookies Collection
		html += "<BR><BR>" + heading.Replace("<!--HEADER-->", "Cookies Collection");
		html += CollectionToHtmlTable(HttpContext.Current.Request.Cookies);
		// Session Variables
		html += "<BR><BR>" + heading.Replace("<!--HEADER-->", "Session Variables");
		html += CollectionToHtmlTable(HttpContext.Current.Session);
		// Server Variables
		html += "<BR><BR>" + heading.Replace("<!--HEADER-->", "Server Variables");
		html += CollectionToHtmlTable(HttpContext.Current.Request.ServerVariables);
		return html;
	}
	static private string CollectionToHtmlTable(NameValueCollection collection) {
		// <TD>...</TD> Template
		const string TD = "<TD><FONT face=\"Arial\" size=\"2\"><!--VALUE--></FONT></TD>";
		// Table Header
		string html = "\n<TABLE width=\"100%\">\n"
			+ " <TR bgcolor=\"#C0C0C0\">" + TD.Replace("<!--VALUE-->", "&nbsp;<B>Name</B>")
			+ " " + TD.Replace("<!--VALUE-->", "&nbsp;<B>Value</B>") + "</TR>\n";
		// No Body? -> N/A
		if (collection.Count == 0) {
			collection = new NameValueCollection();
			collection.Add("N/A", "");
		}
		// Table Body
		for (int i = 0; i < collection.Count; i++) {
			html += "<TR valign=\"top\" bgcolor=\"" + ((i % 2 == 0) ? "white" : "#EEEEEE") + "\">"
				+ TD.Replace("<!--VALUE-->", collection.Keys[i]) + "\n"
				+ TD.Replace("<!--VALUE-->", collection[i]) + "</TR>\n";
		}
		// Table Footer
		return html + "</TABLE>";
	}
	static private string CollectionToHtmlTable(HttpCookieCollection collection) {
		// Overload for HttpCookieCollection collection.
		// Converts HttpCookieCollection to NameValueCollection
		NameValueCollection NVC = new NameValueCollection();
		foreach (string item in collection) NVC.Add(item, collection[item].Value);
		return CollectionToHtmlTable(NVC);
	}
	static private string CollectionToHtmlTable(System.Web.SessionState.HttpSessionState collection) {
		// Overload for HttpSessionState collection.
		// Converts HttpSessionState to NameValueCollection
		NameValueCollection NVC = new NameValueCollection();
		foreach (string item in collection) NVC.Add(item, collection[item].ToString());
		return CollectionToHtmlTable(NVC);
	}
	static private string cleanHTML(string Html) {
		// Cleans the string for HTML friendly display
		return (Html.Length == 0) ? "" : Html.Replace("<", "&lt;").Replace("\r\n", "<BR>").Replace("&", "&amp;").Replace(" ", "&nbsp;");
	}
}

