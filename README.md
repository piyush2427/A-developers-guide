# A-developers-guide
js and other sample codes


=================================================================================================================================
	# form validator to validate any form whith values (On Submit)
	=================================================================================================================================
	
	<script type="text/javascript">
	new FormValidator('userGrpUserMapCRUDForm',
		[{
			name : 'userGroup',
			display : 'User Group',
			actions : 'required'
		}, {
			name : 'loginId',
			display : 'Login Id',
			actions : 'required'
		},
		
		],
		function(errors, evt) {
			if (errors.length > 0) {
				var errorString = '';
				document.getElementById("idErrTbl").style.display = "inline";
				for (var i = 0, errorLength = errors.length; i < errorLength; i++) {
					errorString += errors[i].message + '<br />';
				}
				document.getElementById("el").innerHTML = errorString;
			}
		});
	</script>
	
	=================================================================================================================================
	# Script to append values in a text area of any input type with text
	=================================================================================================================================
	
	<script>

function addLeftBracket(){
	$('#ruleQuery').val($('#ruleQuery').val() +"( ");
}
function addRightBracket(){
	$('#ruleQuery').val($('#ruleQuery').val() +") ");
}
function addAnd(){
	$('#ruleQuery').val($('#ruleQuery').val() +"AND ");
}
function addOr(){
	$('#ruleQuery').val($('#ruleQuery').val() +"OR ");
}
function addOperators(){
	var Operators = $("#operators option:selected").text();
	var leftField = $("#leftField option:selected").val();
	var rightField= $('#rightField').val();
	$('#ruleQuery').val($('#ruleQuery').val() +leftField+" "+Operators+" "+rightField+" ");
}
</script>

=================================================================================================================================
# Jfree Charts ChartDisplayAction.java 
=================================================================================================================================
package com.infrasofttech.omning.action.lcs;

import java.awt.Color;
import java.text.DecimalFormat;
import java.util.ArrayList;
import java.util.Calendar;
import java.util.Date;
import java.util.HashMap;
import java.util.List;
import java.util.Random;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.apache.log4j.Logger;
import org.apache.struts2.ServletActionContext;
import org.apache.struts2.interceptor.ServletRequestAware;
import org.jfree.chart.ChartFactory;
import org.jfree.chart.ChartPanel;
import org.jfree.chart.JFreeChart;
import org.jfree.chart.axis.NumberAxis;
import org.jfree.chart.labels.PieSectionLabelGenerator;
import org.jfree.chart.labels.StandardCategoryItemLabelGenerator;
import org.jfree.chart.labels.StandardPieSectionLabelGenerator;
import org.jfree.chart.plot.CategoryPlot;
import org.jfree.chart.plot.PiePlot;
import org.jfree.chart.plot.Plot;
import org.jfree.chart.plot.PlotOrientation;
import org.jfree.chart.renderer.category.BarRenderer3D;
import org.jfree.data.category.DefaultCategoryDataset;
import org.jfree.data.general.DefaultPieDataset;

import com.infrasofttech.beans.OmniWebTO;
import com.infrasofttech.domain.entities.AccessObject;
import com.infrasofttech.domain.entities.MenuMst;
import com.infrasofttech.domain.entities.lcs.CaseMst;
import com.infrasofttech.domain.entities.lcs.ChartMst;
import com.infrasofttech.domain.entities.lcs.LawFirmLawyerDetail;
import com.infrasofttech.domain.entities.lcs.LawFirmMst;
import com.infrasofttech.domain.entities.lcs.MessageMst;
import com.infrasofttech.domain.entities.lcs.PTPMst;
import com.infrasofttech.domain.entities.lcs.QueueMst;
import com.infrasofttech.omning.services.lcs.CaseMstService;
import com.infrasofttech.omning.services.lcs.LawFirmLawyerDetailService;
import com.infrasofttech.omning.services.lcs.LawFirmMstService;
import com.infrasofttech.omning.services.lcs.MessageMstService;
import com.infrasofttech.omning.services.lcs.PTPMstService;
import com.infrasofttech.omning.services.lcs.QueueMstService;
import com.infrasofttech.omning.utils.SpringUtil;
import com.infrasofttech.utils.OmniConstants;
import com.opensymphony.xwork2.ActionContext;
import com.opensymphony.xwork2.ActionSupport;

public class ChartMstDisplayAction extends ActionSupport implements ServletRequestAware {

	private static final long serialVersionUID = -1L;
	private static final Logger logger = Logger.getLogger(ChartMstDisplayAction.class);
	private HttpServletRequest request =(HttpServletRequest) ActionContext.getContext().get(ServletActionContext.HTTP_REQUEST);//
	private ChartMst chartMst;
	private String mode = "";
	private String errMsg = "";
	private boolean flag = false;
	private MenuMst menuMst;

	private Long refPKId = -1L;
	private boolean isActive;
	private OmniWebTO omniWebTO = new OmniWebTO();
	private Long id;
	HttpSession session = null;	
	private JFreeChart chart;
	
	public HashMap<String , String> ruleForPtp(){ // rule for PTP for a Bar chart
		PTPMstService ptpMstService = (PTPMstService) SpringUtil
				.getSpringUtil().getService("ptpMstService");
		List<PTPMst> ptpMsts = ptpMstService.getPTPMstListByTenant(getAccessObject().getTenantId(), OmniConstants.AUTH_AUTHORIZED, true); 
		HashMap<String , String> ptpChartMap= new HashMap<String , String>();
		Integer XLBL1value= 0;//for Broken Ptp
		Integer XLBL2value=0;//for WIP
		for(PTPMst ptp :ptpMsts)
		{
			if(ptp.getStatus().equalsIgnoreCase("BROKEN"))
					{
				XLBL1value++;
				
					}
			else if(ptp.getStatus().equalsIgnoreCase("WIP"))
			{
				XLBL2value++;
			}
				
		}
	
		Integer XLBL3value= ptpMsts.size() ;
		
		ptpChartMap.put("Xlabels", "Broken PTP,WIP PTP,Total PTP");
		ptpChartMap.put("Xaxislabel", "Promise to Pay(PTP)");
		
		ptpChartMap.put("Yaxislabel", "No of PTP");
		ptpChartMap.put("Title", "Promise to Pay");
		ptpChartMap.put("XLBL1value", XLBL1value.toString());//Broken PTP
		ptpChartMap.put("XLBL2value", XLBL2value.toString());// WIP PTP
		ptpChartMap.put("XLBL3value",XLBL3value.toString());// TOTAL PTP
		return ptpChartMap;
	}
	
	
	public HashMap<String , String> ruleForOverdue(){// Rule for Overdue of loan accounts for a bar chart
		PTPMstService ptpMstService = (PTPMstService) SpringUtil
				.getSpringUtil().getService("ptpMstService");
		List<PTPMst> ptpMsts = ptpMstService.getPTPMstListByTenant(getAccessObject().getTenantId(), OmniConstants.AUTH_AUTHORIZED, true); 
		HashMap<String , String> ptpChartMap= new HashMap<String , String>();
		ArrayList<String> loanAcc = new ArrayList<String>();
		ArrayList<String>overdueOfLoanAcc = new ArrayList<String>();
		int i=0;
		for(PTPMst ptp :ptpMsts)
		{
			loanAcc.add(ptp.getLoanAcNo());
			overdueOfLoanAcc.add(ptp.getOverdueAmount().toString());
			
		}
		String Xlabels=loanAcc.toString().replace("[", "").replace("]", "");
		ptpChartMap.put("Xlabels", Xlabels);
		ptpChartMap.put("Xaxislabel", "Loan Accounts");
		
		ptpChartMap.put("Yaxislabel", "Total Overdue by Loan Account No");
		ptpChartMap.put("Title", "Loan Account Overdue");
		for(int j=1;j<=loanAcc.size();j++){
		ptpChartMap.put("XLBL"+j+"value",overdueOfLoanAcc.get(j-1) );
		
		}
		return ptpChartMap;
	}
	
	public HashMap<String , String> ruleForLawfirm(){// Rule for LawFirm for a bar chart
		LawFirmLawyerDetailService lawFirmLawyerDetailService = (LawFirmLawyerDetailService) SpringUtil
				.getSpringUtil().getService("lawFirmLawyerDetailService");
		LawFirmMstService lawFirmMstService =(LawFirmMstService) SpringUtil
				.getSpringUtil().getService("lawFirmMstService");

		
		List<LawFirmMst> lawFirmMsts = lawFirmMstService.getLawFirmMstListByTenant(getAccessObject().getTenantId(), OmniConstants.AUTH_AUTHORIZED, true);
		HashMap<String , String> ptpChartMap= new HashMap<String , String>();
		ArrayList<String> lawFirms = new ArrayList<String>();
		Integer countLawers=0;
		ArrayList<String> lawyers = new ArrayList<String>();
	
		for(LawFirmMst lawFirmMst :lawFirmMsts)
		{
			countLawers=0;
			lawFirms.add(lawFirmMst.getLawFirmName());
			List<LawFirmLawyerDetail> lawFirmLawyerDetails = lawFirmLawyerDetailService.getLawFirmLawyerDetailListByTenant(getAccessObject().getTenantId(), OmniConstants.AUTH_AUTHORIZED, true,lawFirmMst.getLawFirmCode()); 
			for(LawFirmLawyerDetail firmLawyerDetail:lawFirmLawyerDetails)
			{
				countLawers++;
				
			}
			lawyers.add(countLawers.toString());
		}
		String Xlabels=lawFirms.toString().replace("[", "").replace("]", "");
		ptpChartMap.put("Xlabels", Xlabels);
		ptpChartMap.put("Xaxislabel", "Law Firm's");
		
		ptpChartMap.put("Yaxislabel", "Total Lawyer's in lawfirm's");
		ptpChartMap.put("Title", "LawFirm v/s Lawyers");
		for(int j=1;j<=lawFirms.size();j++){
		ptpChartMap.put("XLBL"+j+"value",lawyers.get(j-1) );
		
		}
		return ptpChartMap;
	}
	
	public HashMap<String , String> ruleForBuckets(){// Rule for buckets for a pie chart
		QueueMstService  queueMstService = (QueueMstService) SpringUtil
				.getSpringUtil().getService("queueMstService");
		List<QueueMst>  queueMsts = queueMstService.getQueueMstListByTenant(getAccessObject().getTenantId(), OmniConstants.AUTH_AUTHORIZED, true); 
		HashMap<String , String> ptpChartMap= new HashMap<String , String>();
		Integer retailCount=0;
		Integer corporateCount=0;
		
		for(QueueMst queue :queueMsts)
		{
			
			if(queue.getCustomerType().equalsIgnoreCase("Retail"))
			{
				retailCount++;
			}
			else if(queue.getCustomerType().equalsIgnoreCase("Corporate"))
			{
				corporateCount++;
			}
			
		}
		String Xlabels="Retail,Corporate";
		ptpChartMap.put("Xlabels", Xlabels);
		ptpChartMap.put("Title", "Buckets(Retails/Corporate)");
		
		ptpChartMap.put("XLBL1value",retailCount.toString() );
		ptpChartMap.put("XLBL2value",corporateCount.toString() );
		
		
		return ptpChartMap;
	}
	
	public HashMap<String , String> ruleForOverdueAcctStatus(){// Rule for overdue Loan Account status for a pie chart
		CaseMstService caseMstService = (CaseMstService) SpringUtil
				.getSpringUtil().getService("caseMstService");
		List<CaseMst> caseMsts  = caseMstService.getCaseMstListByTenant(getAccessObject().getTenantId(), true); 
		HashMap<String , String> ptpChartMap= new HashMap<String , String>();
		Integer closedCount=0;
		Integer totalCount=caseMsts.size() ;
		ArrayList<String> loanAcc = new ArrayList<String>();
		ArrayList<String>overdueOfLoanAcc = new ArrayList<String>();
		Calendar cal = Calendar.getInstance();
		cal.add(Calendar.MONTH, -12);
		Date result = cal.getTime();
		for(CaseMst cases :caseMsts)
		{
			if(cases.getCaseStatus().equalsIgnoreCase("CLOSE"))
			{
				if(cases.getLastModifiedDate().after(result))
				closedCount++;
			}
			
			
		}
		String Xlabels="Total Overdue accounts reported,Closed O/D accounts";
		ptpChartMap.put("Xlabels", Xlabels);
		ptpChartMap.put("Title", "Total OD accounts reported v/s Closed in last 12 months");
		
		ptpChartMap.put("XLBL1value",totalCount.toString() );
		ptpChartMap.put("XLBL2value",closedCount.toString() );
		
		
		return ptpChartMap;
	}
	public HashMap<String , String> ruleForMessageFreq(){// Rule for Message Frequency for a pie chart
		
		MessageMstService messageMstService = (MessageMstService) SpringUtil
				.getSpringUtil().getService("messageMstService");
		List<MessageMst>   messageMsts  = messageMstService.getMessageMstListByTenant(getAccessObject().getTenantId(),OmniConstants.AUTH_AUTHORIZED, true); 
		HashMap<String , String> ptpChartMap= new HashMap<String , String>();
		Integer typeCount1=0;
		Integer typeCount2=0;
		Integer typeCount3=0;
		Integer typeCount4=0;
		Integer typeCount5=0;

		ArrayList<String>messageFreq = new ArrayList<String>();
		for(MessageMst messages :messageMsts)
		{
			
			String x=messages.getFrequency();
			if(x.equalsIgnoreCase("DAILY")){
				typeCount1++;
				
			}
			if(x.equalsIgnoreCase("MONTHLY")){
				typeCount2++;
				
			}
			if(x.equalsIgnoreCase("HOURLY")){
				typeCount3++;
			
			}
			if(x.equalsIgnoreCase("ONE-TIME")){
				typeCount4++;
				
			}
			if(x.equalsIgnoreCase("WEEKLY")){
				typeCount5++;
				
			}	
		}
		
		messageFreq.add(typeCount1.toString());
		messageFreq.add(typeCount2.toString());
		messageFreq.add(typeCount3.toString());
		messageFreq.add(typeCount4.toString());
		messageFreq.add(typeCount5.toString());
		String Xlabels="DAILY,MONTHLY,HOURLY,ONE-TIME,WEEKLY";
		ptpChartMap.put("Xlabels", Xlabels);
		ptpChartMap.put("Title", "Message Frequency V/s Messages Category");
		
		for(int j=1;j<=5;j++){
			ptpChartMap.put("XLBL"+j+"value",messageFreq.get(j-1) );
		}
		
		return ptpChartMap;
	}
	
	public String drawPIEChart()
	{
		HashMap<String , String> chartMap= new HashMap<String , String>();
		
		String pieRule=request.getParameter("pieRule");
		if(pieRule.equalsIgnoreCase("BUCKETS"))
		{
		chartMap=ruleForBuckets();
		}
		else if(pieRule.equalsIgnoreCase("OVERDUECLOSED"))
		{
			chartMap=ruleForOverdueAcctStatus();
		}
		else if(pieRule.equalsIgnoreCase("MESSAGEFREQ"))
		{
			chartMap=ruleForMessageFreq();
		}
		String[] Xlabels=chartMap.get("Xlabels").split(",");
		int noOfBars=Xlabels.length;
		
     DefaultPieDataset dataSet = new DefaultPieDataset();
     
     for(int i=1; i<=noOfBars;i++)
	 {
		 
		 Double val=Double.parseDouble(chartMap.get("XLBL"+i+"value")); 
		 dataSet.setValue(Xlabels[i-1],val);
	  
	 }

    List<DefaultPieDataset> defaultPieDatasets=new ArrayList<DefaultPieDataset>();
    defaultPieDatasets =dataSet.getKeys();
    int size=defaultPieDatasets.size();
    chart = ChartFactory.createPieChart3D(
    		chartMap.get("Title"), // Title
            dataSet,                    // Data
            true,                       // Display the legend
            true,                       // Display tool tips
            false                       // No URLs
            );

    chart.setBorderVisible(false); 
   chart.setBackgroundPaint(Color.WHITE);
    ChartPanel chartPanel = new ChartPanel( chart ); 
    chartPanel.setForeground(Color.white);
    chartPanel.setBackground(Color.white);
	PiePlot plot = (PiePlot) chart.getPlot();
    plot.setNoDataMessage("No data available");
    plot.setCircular(false);
    plot.setLabelGap(0.02);
    PieSectionLabelGenerator gen = new StandardPieSectionLabelGenerator(
            "{0}: {1} ({2})", new DecimalFormat("0"), new DecimalFormat("0%"));
        plot.setLabelGenerator(gen);
    plot.setSimpleLabels(true);
    Random rand = new Random();
    Color randomColor= new Color(rand.nextFloat(), rand.nextFloat(), rand.nextFloat()) ;
    for(int i=0; i<size ;i++){
    randomColor= new Color(rand.nextFloat(), rand.nextFloat(), rand.nextFloat());
    plot.setSectionPaint(i, randomColor);}
    return SUCCESS;
    }
    
	
public String drawBARChart()
{
	HashMap<String , String> chartMap= new HashMap<String , String>();
	
	String barRule=request.getParameter("barRule");
	if(barRule.equalsIgnoreCase("PTP"))
	{
	chartMap=ruleForPtp();
	}
	else if(barRule.equalsIgnoreCase("OVERDUE"))
	{
		chartMap=ruleForOverdue();
	}
	else if(barRule.equalsIgnoreCase("LAWFIRMLAWER"))
	{
		chartMap=ruleForLawfirm();
	}
	
	String[] Xlabels=chartMap.get("Xlabels").split(",");
	int noOfBars=Xlabels.length;
		 DefaultCategoryDataset barDataset = new DefaultCategoryDataset();
		 for(int i=1; i<=noOfBars;i++)
		 {
			 
			 Double val=Double.parseDouble(chartMap.get("XLBL"+i+"value")); 
		    barDataset.setValue(val,"", Xlabels[i-1]);
		  
		 }
		    //Create the chart
		    chart = ChartFactory.createBarChart3D(
		    		chartMap.get("Title"), chartMap.get("Xaxislabel"), chartMap.get("Yaxislabel"), barDataset,
		        PlotOrientation.VERTICAL, false, true, false);
		    
		    chart.setBorderVisible(false); 
		    chart.setBackgroundPaint(Color.white);
			      ChartPanel chartPanel = new ChartPanel( chart );        
			      chartPanel.setPreferredSize(new java.awt.Dimension( 580 , 167 ) );        
			      CategoryPlot categoryPlot = chart.getCategoryPlot();
			     
			      categoryPlot.setBackgroundPaint(Color.white);
			      BarRenderer3D br = (BarRenderer3D) categoryPlot.getRenderer();
			      br.setMaximumBarWidth(0.1);
			      final NumberAxis rangeAxis = (NumberAxis) categoryPlot.getRangeAxis();
	              rangeAxis.setStandardTickUnits(NumberAxis.createIntegerTickUnits());
	              Plot plot = chart.getPlot();
	              BarRenderer3D barRenderer = (BarRenderer3D)((CategoryPlot) plot).getRenderer();
			      
	              Random rand = new Random();
			         float r = rand.nextFloat();
			         float g = rand.nextFloat();
			         float b = rand.nextFloat();
			         Color randomColor = new Color(r, g, b);
			         
			         barRenderer.setSeriesPaint(0, randomColor);
			         DecimalFormat decimalformat1 = new DecimalFormat("##,###.00");
			         barRenderer.setSeriesItemLabelGenerator(0, new StandardCategoryItemLabelGenerator("{2}", decimalformat1)); //i added your line here.        
			         barRenderer.setSeriesItemLabelsVisible(0, true);
			         barRenderer.setSeriesItemLabelPaint(0, Color.black);			        
			         barRenderer.setSeriesItemLabelsVisible(0,true);
			         chart.getCategoryPlot().setRenderer(barRenderer);
			         
		       
			         return SUCCESS;
}
 // This method will get called if we specify <param name="value">chart</param>
 public JFreeChart getChart() {
    return chart;
 }
			
            
       


	private AccessObject getAccessObject() {
		return (AccessObject) request.getSession(false).getAttribute(
			OmniConstants.SESSION_PARAM_ACCESSOBJECT);
	}

	public void setServletRequest(HttpServletRequest arg0) {
		this.request = arg0;
	}


	private HttpServletResponse getResponse() {
		return ServletActionContext.getResponse();
	}
}

=================================================================================================================================
# Jfree Charts Struts integration 
=================================================================================================================================
//maven dependencies: pom.xml
<dependency>
			<groupId>org.jfree</groupId>
			<artifactId>jfreechart</artifactId>
			<version>1.0.19</version>
		</dependency>
		<dependency>
		    <groupId>jfree</groupId>
		    <artifactId>jcommon</artifactId>
		    <version>1.0.15</version>
    	</dependency>
    	<dependency>
			   <groupId>org.apache.struts</groupId>
			   <artifactId>struts2-jfreechart-plugin</artifactId>
			   <version>2.0.11</version>
		</dependency>

<package name="default" namespace="/" extends="jfreechart-default">
       <action name="drawChart_*" method="{1}" class="com.infrasofttech.omning.action.lcs.ChartMstDisplayAction">
            <result name="success" type="chart">
                <param name="value">chart</param>
                <param name="type">jpeg</param>
                <param name="width">700</param>
                <param name="height">500</param>
            </result>
        </action>
        
   </package>
   =================================================================================================================================
  # Rule evaluation i.e Expression evaluation using ScriptEngine
   =================================================================================================================================
   
   public String getRuleForOverdue(String tenantId,String overdueAmount, String overdueDays,String overdueInstalments) throws OmniNGException{
		String qId="";
		if (null == tenantId) {
			throw new RuntimeException("RuleMstService : query params CANNOT be null");
		}
		Map<String, Object> queryParams = new HashMap<String, Object>();
		queryParams.put("tenantId", tenantId);
		queryParams.put("isActive", true); 
		queryParams.put("authStatus", OmniConstants.AUTH_AUTHORIZED); 
		
		List<RuleMst>  ruleMstList = executeNamedQuery("RuleMst.getRuleMstList", queryParams);
		//TODO parse through the rules and get the rule's queue id
		for(RuleMst ruleMst:ruleMstList)
		{
			
			
			try{
				if(ruleMst.getRuleType().equalsIgnoreCase("functional"))
				{
					String ruleQuery=ruleMst.getRuleQuery();
					ruleQuery=ruleQuery.replace("overdueAmount", overdueAmount).replace("overdueDays", overdueDays).replace("overdueInstalments", overdueInstalments);
					ruleQuery=ruleQuery.replace("AND", "&&").replace("OR", "||").replace("=", "==");
				
					ScriptEngineManager sem = new ScriptEngineManager();
					ScriptEngine se = sem.getEngineByName("JavaScript");
		            String myExpression = ruleQuery;
		            Boolean x=(Boolean) se.eval(myExpression);
		            System.out.println("ruleQuery========="+ruleQuery);
		            System.out.println("reuslt********************"+se.eval(myExpression)+"******x="+x);
		        
		            if(x==true)
		            {
		            	qId=ruleMst.getqId();
		            	break;
		            }
		            
				}
			}
			catch (ScriptException e) {

	            System.out.println("Invalid Expression");
	            e.printStackTrace();

	        }
		}
		return qId;
	}
	
	 =================================================================================================================================
	 # Date Picker Rnd Code pen
	 =================================================================================================================================
	 <style class="cp-pen-styles">label{margin-left: 20px;}
	#datepicker{width:180px; margin: 0 20px 20px 20px;}
	#datepicker > span:hover{cursor: pointer;}
	</style>
	 <input id="startDate" class="col-md-8 input-sm datepicker_multi" name="startDate" type="text" style="background-color: lightyellow" 
					 value="<fmt:formatDate pattern="<%=DateUtil.DATE_FORMAT_DDMMYYYY_HHMMSS%>" value="<%=messageMst.getStartDate()%>"/>"  />
	 <script>
	 
	$(document).ready(function() {
		 var mode=$("#mode").val();
		 if(mode=="CREATE"){
		loadActionTypeAndSubType();}
		var day=$("#todayDate").val();
		var vToken=$("#vToken").val();
		var today =new Date(day);
		$('.datepicker_multi').each(function(){
            $(this).datepicker({
                changeMonth : true,
                changeYear : true,
                yearRange : '1960:2100',
                defaultDate : today
            });         
            var id=$(this).attr('id');                  
            $("#"+id).change(function(){   
				var effFromDate=$("#"+id).val();
				var dateFormat = $('#dateFormat').val();
				$.post("dateFormatter", {
					effFromDate : effFromDate,
					dateFormat: dateFormat,
					vToken : vToken
				}, function(result) {
					if(id.trim()=="startDate"){
						var starttime = "00" + ":" + "00" + ":" + "00" + ":" + "000";
					document.getElementById(id).value=result+" "+starttime;
					}
					else if(id=="endDate"){
						var endtime = "23" + ":" + "59" + ":" + "59" + ":" + "999";
						document.getElementById(id).value=result+" "+endtime;
					}
				});
			
            }); 
        });
	});
</script>

 =================================================================================================================================
 # Overdue calculation logic by me
 =================================================================================================================================
 package com.infrasofttech.omning.action.lcs;

import java.text.ParseException;
import java.util.ArrayList;
import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.apache.log4j.Logger;
import org.apache.struts2.ServletActionContext;
import org.apache.struts2.interceptor.ServletRequestAware;

import com.infrasofttech.beans.OmniWebTO;
import com.infrasofttech.domain.entities.AccessObject;
import com.infrasofttech.domain.entities.MenuMst;
import com.infrasofttech.domain.entities.lcs.CaseMst;
import com.infrasofttech.domain.entities.lcs.OverdueProcessDetails;
import com.infrasofttech.domain.entities.lcs.OverdueProcessVO;
import com.infrasofttech.exceptions.OmniNGException;
import com.infrasofttech.omning.services.lcs.CaseMstService;
import com.infrasofttech.omning.services.lcs.OverdueProcessDetailsService;
import com.infrasofttech.omning.services.lcs.RuleMstService;
import com.infrasofttech.omning.utils.OmniNGServiceFactory;
import com.infrasofttech.omning.utils.SpringUtil;
import com.infrasofttech.utils.OmniConstants;
import com.opensymphony.xwork2.ActionContext;
import com.opensymphony.xwork2.ActionSupport;

public class OverdueProcessDetailsCRUDAction extends ActionSupport implements ServletRequestAware {

	private static final long serialVersionUID = -1L;
	private static final Logger logger = Logger.getLogger(OverdueProcessDetailsCRUDAction.class);
	private HttpServletRequest request = (HttpServletRequest) ActionContext.getContext()
			.get(ServletActionContext.HTTP_REQUEST);//
	private OverdueProcessDetails overdueProcessDetails;
	private CaseMst caseMst;
	private String mode = "";
	private String errMsg = "";
	private boolean flag = false;
	private MenuMst menuMst;
	private String actionCode = "";
	private Long refPKId = -1L;
	private boolean isActive;
	private OmniWebTO omniWebTO = new OmniWebTO();
	private Long id;
	HttpSession session = null;

	public String saveOverdueProcessDetails() throws Exception {
		String retVal = OmniConstants.LOGIN;
		Boolean caseSaveStatus = false;
		try {
			actionCode = OmniConstants.UPDATE;
			mode = (String) request.getParameter("mode");
			menuMst = (MenuMst) getAccessObject().getMenuMst();
			// sample webservice data
			OverdueProcessVO chkOverdue1 = new OverdueProcessVO();
			chkOverdue1.setCustomerCode("C01");
			chkOverdue1.setLoanAcNo("LOANACCNO1122335777");
			chkOverdue1.setOverdueAmount("4000");
			chkOverdue1.setOverdueCurrency("RS");
			chkOverdue1.setOverdueDays("80");
			chkOverdue1.setOverdueInstalments("12");
			OverdueProcessVO chkOverdue2 = new OverdueProcessVO();
			chkOverdue2.setCustomerCode("C10");
			chkOverdue2.setLoanAcNo("LOANACCNO223377333");
			chkOverdue2.setOverdueAmount("8976");
			chkOverdue2.setOverdueCurrency("RS");
			chkOverdue2.setOverdueDays("50");
			chkOverdue2.setOverdueInstalments("6");
			// -----------------
			List<OverdueProcessVO> overdueProcessVOs = new ArrayList<OverdueProcessVO>();
			overdueProcessVOs.add(chkOverdue1);
			overdueProcessVOs.add(chkOverdue2);
			for (OverdueProcessVO chkOverdue : overdueProcessVOs) {
				
				CaseMstService caseMstService = (CaseMstService) SpringUtil.getSpringUtil()
						.getService("caseMstService");
				List<CaseMst> caseMstList = (List<CaseMst>) caseMstService
						.getCaseMstListByTenant(getAccessObject().getTenantId(), true);

				overdueProcessDetails = setValuesToOverdueProcessDetails(overdueProcessDetails, chkOverdue);
				overdueProcessDetails.setTenantId(getAccessObject().getTenantId());
				overdueProcessDetails = getOverdueProcessDetailsService().saveOrUpdate(getAccessObject().getLoginId(),
						overdueProcessDetails);

				if (null != overdueProcessDetails.getCaseNo() && !overdueProcessDetails.getCaseNo().isEmpty()) {
					if (caseMstList.isEmpty()) {
						caseMst = setValuesToCaseMst(caseMst, overdueProcessDetails);
						caseMst.setTenantId(getAccessObject().getTenantId());
						// Save The Case
						caseMst = caseMstService.saveOrUpdate(getAccessObject().getLoginId(), caseMst);
					}

					for (CaseMst cases : caseMstList) {
						if (!cases.getCaseNo().equalsIgnoreCase(overdueProcessDetails.getCaseNo())) {
							caseSaveStatus = true;
						} else {
							caseSaveStatus = false;
							break;
						}
					}
					if (caseSaveStatus == true) {
						caseMst = setValuesToCaseMst(caseMst, overdueProcessDetails);
						caseMst.setTenantId(getAccessObject().getTenantId());
						// Save The Case
						caseMst = caseMstService.saveOrUpdate(getAccessObject().getLoginId(), caseMst);
					}
				}
			}
			if (null != overdueProcessDetails) {
				retVal = OmniConstants.SUCCESS;
				refPKId = overdueProcessDetails.getId();
				errMsg = "Overdue Process Executed Successfully !";
			}
			request.setAttribute(OmniConstants.REQUEST_INPUTOBJ, overdueProcessDetails);
		} catch (OmniNGException oe) {//
			logger.error("", oe);
			omniWebTO.setErrorCode(oe.getMessage());
		
			if (errMsg == null || errMsg == "") {
				errMsg = "Error In Operation";
			}
			retVal = OmniConstants.INPUT;
		} catch (Exception e) {//
			errMsg = "Error In Operation";
			logger.error("", e);
			omniWebTO.setErrorCode(e.getMessage());
			retVal = OmniConstants.INPUT;
		} finally {//
			request.setAttribute("type", OmniConstants.USERVIEW);
			request.setAttribute("errMsg", errMsg);
			omniWebTO.setActionCode(actionCode);
			omniWebTO.setRefPKId(refPKId);
			request.setAttribute(OmniConstants.REQUEST_OMNIWEBTO, omniWebTO);
		}
		return retVal;
	}

	public String getOverdueProcessDetails()throws Exception {
		String retVal = OmniConstants.LOGIN;
		try {
		
			if (null != request.getParameter("id") ) {
				mode = OmniConstants.UPDATE;
				id = Long.parseLong(request.getParameter("id"));
				overdueProcessDetails = (OverdueProcessDetails) getOverdueProcessDetailsService().find(id);
			} else {
				if (null != request.getParameter("id")) {
					id = Long.parseLong(request.getParameter("id"));
				}
				else{
					id = -1L;
				}
				OverdueProcessDetails entity = (OverdueProcessDetails) getOverdueProcessDetailsService().find(id);
				if (null == entity) {
					overdueProcessDetails  =  new OverdueProcessDetails();
					mode = OmniConstants.CREATE;//
				} else {
					overdueProcessDetails  =  entity;
				}
			}
			request.setAttribute(OmniConstants.REQUEST_INPUTOBJ, overdueProcessDetails);
			request.setAttribute("type", request.getParameter("type"));
			request.setAttribute("mode", mode);
			request.setAttribute("flag", flag);
			retVal = OmniConstants.SUCCESS;
		} catch (Exception e) {
			logger.error("", e);
			omniWebTO.setErrorCode(e.getMessage());
			retVal = OmniConstants.INPUT;
		} finally {
			omniWebTO.setActionCode(OmniConstants.VIEW);
			omniWebTO.setRefPKId(refPKId);
			request.setAttribute(OmniConstants.REQUEST_OMNIWEBTO, omniWebTO);
		}
		return retVal;
	}

	private OverdueProcessDetails setValuesToOverdueProcessDetails(OverdueProcessDetails overdueProcessDetails,
			OverdueProcessVO chkOverdue) throws ParseException {

		try {
			String casetype = "";
			String caseNo = "";
			String queueId="";
			overdueProcessDetails = new OverdueProcessDetails();

			OverdueProcessDetailsService overdueProcessDetailsService = (OverdueProcessDetailsService) SpringUtil
					.getSpringUtil().getService("overdueProcessDetailsService");
			List<OverdueProcessDetails> overdueProcessDetailsList = overdueProcessDetailsService
					.getOverdueProcessDetailsListByTenant(getAccessObject().getTenantId(), true);
			
			overdueProcessDetails.setOverdueAmount(chkOverdue.getOverdueAmount());
			overdueProcessDetails.setOverdueCurrency(chkOverdue.getOverdueCurrency());
			overdueProcessDetails.setOverdueDays(chkOverdue.getOverdueDays());
			overdueProcessDetails.setOverdueInstalments(chkOverdue.getOverdueInstalments());
			
			//TODO assign Queue to overdueProcessDetails
			RuleMstService ruleMstService =(RuleMstService)SpringUtil
					.getSpringUtil().getService("ruleMstService");
			
			queueId = ruleMstService.getRuleForOverdue(getAccessObject().getTenantId(),chkOverdue.getOverdueAmount()
					,chkOverdue.getOverdueDays(),chkOverdue.getOverdueInstalments());
		
				if(null==queueId || queueId.isEmpty() )
				{
					
					errMsg ="No Rule defined for Loan Ac - "+chkOverdue.getLoanAcNo();
				}
			
			
			overdueProcessDetails.setqId(queueId);
			if (overdueProcessDetailsList.isEmpty()) {
				// 1st entry in table
				overdueProcessDetails.setCaseStatus("OPEN");
				overdueProcessDetails.setUpdateDate(getAccessObject().getOprtnDate());
				overdueProcessDetails.setCustomerCode(chkOverdue.getCustomerCode());
				overdueProcessDetails.setLoanAcNo(chkOverdue.getLoanAcNo());
			}
			for (OverdueProcessDetails overdue : overdueProcessDetailsList) {
				if (!chkOverdue.getLoanAcNo().equalsIgnoreCase(overdue.getLoanAcNo()) == true) {
					// Create a new case
					casetype = "new";

				} else {
					// case already exists
					casetype = "alreadyExists";
					caseNo = overdue.getCaseNo();
					//TODO close a case 
					if(overdue.getOverdueAmount().equalsIgnoreCase("0"))
					{
						casetype="alreadyExistsAndClosed";
					}
					break;
				}
			}

			if (casetype.equalsIgnoreCase("new")) {
				overdueProcessDetails.setCaseStatus("OPEN");
				overdueProcessDetails.setUpdateDate(getAccessObject().getOprtnDate());
				overdueProcessDetails.setCustomerCode(chkOverdue.getCustomerCode());
				overdueProcessDetails.setLoanAcNo(chkOverdue.getLoanAcNo());
			} else if (casetype.equalsIgnoreCase("alreadyExists")) {
				overdueProcessDetails.setCaseNo(caseNo);
				overdueProcessDetails.setCaseStatus("OPEN");
				overdueProcessDetails.setCustomerCode(chkOverdue.getCustomerCode());
				overdueProcessDetails.setLoanAcNo(chkOverdue.getLoanAcNo());
				overdueProcessDetails.setUpdateDate(getAccessObject().getOprtnDate());
			}
			else if (casetype.equalsIgnoreCase("alreadyExistsAndClosed")) {
				overdueProcessDetails.setCaseNo(caseNo);
				overdueProcessDetails.setCaseStatus("CLOSE");
				overdueProcessDetails.setCustomerCode(chkOverdue.getCustomerCode());
				overdueProcessDetails.setLoanAcNo(chkOverdue.getLoanAcNo());
				overdueProcessDetails.setUpdateDate(getAccessObject().getOprtnDate());
			}

			return overdueProcessDetails;
		} catch (OmniNGException e) {
			logger.error("", e);
			logger.error("", e);
		}
		return overdueProcessDetails;
	}

	private CaseMst setValuesToCaseMst(CaseMst caseMst, OverdueProcessDetails overdueProcessDetails)
			throws ParseException {

		try {

			caseMst = new CaseMst();
			caseMst.setCaseNo(overdueProcessDetails.getCaseNo());
			caseMst.setCaseStatus(overdueProcessDetails.getCaseStatus());
			caseMst.setLoanAcNo(overdueProcessDetails.getLoanAcNo());
			caseMst.setCustomerCode(overdueProcessDetails.getCustomerCode());
			caseMst.setAuthStatus(OmniConstants.AUTH_AUTHORIZED);
			return caseMst;
		} catch (OmniNGException e) {
			logger.error("", e);
			logger.error("", e);
		}
		return caseMst;
	}

	private List<OverdueProcessDetails> getOverdueProcessDetailsList(String tenantId, String authStatus,
			Boolean isActive) {
		return getOverdueProcessDetailsService().getOverdueProcessDetailsListByTenant(tenantId, isActive);
	}

	private OverdueProcessDetailsService getOverdueProcessDetailsService() {
		return (OverdueProcessDetailsService) OmniNGServiceFactory
				.getService(OmniConstants.MENUCODE_OVERDUEPROCESSDETAILS);
	}

	private AccessObject getAccessObject() {
		return (AccessObject) request.getSession(false).getAttribute(OmniConstants.SESSION_PARAM_ACCESSOBJECT);
	}

	public void setServletRequest(HttpServletRequest arg0) {
		this.request = arg0;
	}

	private HttpServletResponse getResponse() {
		return ServletActionContext.getResponse();
	}
}

===============================================================================================================================
# Higly configurable table Structure using jquery and JSON
===============================================================================================================================
<%@page import="java.util.*"%>
<%@page import="com.infrasofttech.domain.entities.AppConfig"%>
<%@page import="com.infrasofttech.domain.entities.AccessObject"%>
<%@page import="com.infrasofttech.domain.entities.RoleMenuActionAccess"%>
<%@page import="com.infrasofttech.utils.MENUACTIONSTATUS"%>
<%@page import="com.infrasofttech.utils.OmniConstants"%>
<%@page import="com.infrasofttech.domain.entities.fm.BudgetWorkforceMst"%>
<%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt"%>
<%@ taglib prefix="s" uri="/struts-tags"%>

<style>
#mandatory {
	color: red 
}
</style>
<style class="cp-pen-styles">label{margin-left: 20px;}
	#datepicker{width:180px; margin: 0 20px 20px 20px;}
	#datepicker > span:hover{cursor: pointer;}
</style>
<style type="text/css">
	.bs-example{
	margin: 20px;
}
</style>
<style>
table.gridtable  {
    border-collapse: collapse;
    width: 90%;
}

th, td {
    text-align: left;
    padding: 8px;
}

tr.a:nth-child(even){background-color: #f2f2f2}
</style>
<link rel="stylesheet" href="ngcss/jquery-ui.css">
<link href="css/stylesheet.css" rel="stylesheet" />
<link href="newcss/datepicker.css" rel="stylesheet" type="text/css" />
<link rel="stylesheet" type="text/css" href="ngcss/ui.jqgrid.css" >
<link rel="stylesheet" type="text/css" href="ngcss/ui.jqgrid-bootstarp.css" >
<link rel="stylesheet" type="text/css" href="ngcss/searchFilter.css" >
<script type="text/javascript" src="js/validate.js"></script>
<script type="text/javascript" src="js/jquery-ui.js"></script>
<script src="js/customFormatter.js"></script>
<script src='newjs/css_live_reload_init.js'></script>
<script>
	$(function () {
		$('#datepicker').datepicker({
			autoclose: true,
			todayHighlight: true
		}).datepicker('update', new Date());
	});
</script>
<%-- <script type="text/javascript">
	$( window ).load(function() {
		var errMsg = document.getElementById("errMsg").value;
		var mode = document.getElementById("mode").value;
		if(errMsg == <%=OmniConstants.SUCCESS%>){
			if(mode == <%=OmniConstants.UPDATE%>){
				alert("Successfully updated");
			} else{
				alert("successfully saved");
			}
		}
	});
</script> --%>

<script>
	 $(document).ready(function(){
		 var type= $("#type").val();
	if(type == 'USERVIEW'){	
		 $('#budgetWorkforceMstCRUDForm :input').attr('disabled', true);
	}
	$('input[id="update"]').prop('disabled', true);
 });
</script>
<script>

function onView(){
	
	var deptCount=$("input[id^='dept']").size(); // to count the no of dept generated dynamically
	var branchCount=$("input[id^='branch']").size();
	var positionCount=$("input[id^='position']").size();
	//var actualPositions=positionCount/(deptCount*branchCount);
	//var posStart=0;
	 //var posEnd=0;
	//alert(actualPositions);
	//alert(branchCount);
	//alert(positionCount);
	budgetType=$("#budgetType").val();
	budgetSubType=$("#budgetSubType").val();
	companyCode=$("#companyCode").val();
	Periodcode=$("#Periodcode").val();
	if(budgetType=="" || budgetSubType=="" || companyCode=="" || Periodcode=="")
		{
			alert("All fields are mandatory, please select!");
		}
	else{
	var dataToBeSent='budgetType='+budgetType+'&budgetSubType='+budgetSubType+'&companyCode='+companyCode+'&Periodcode='+Periodcode;
	$.ajax({
		  type: 'POST',
		  url: 'getBudgetWorkforceMstAjaxList.action',
		  data: {
			  budgetType : budgetType,
			  budgetSubType : budgetSubType,
			  companyCode : companyCode,
			  Periodcode : Periodcode
		  },
		  success: function(data) 
			{
				var aa=data;	
				//alert(aa);
				var jsonStr=JSON.stringify(data);
				if(null==aa)
					{
					
					 for(j=1;j<=positionCount;j++)
						{
							empPrevYear=$("#empPrevYear"+j).val("");
							empExisting=$("#empExisting"+j).val("");
							empVacancy=$("#empVacancy"+j).val("");
						}
					 alert("No such budget defined!");
					 $('input[id="update"]').prop('disabled', false);
					}
				//alert(aa.length);
				else if (jsonStr.indexOf("STAGE3") >= 0)
					{
					alert("This budget has been approved already, it cannot be modified now!");
					 for(j=1;j<=positionCount;j++)
						{
					$("#empPrevYear"+j).attr("disabled", "disabled"); 	
					$("#empExisting"+j).attr("disabled", "disabled");	
					$("#empVacancy"+j).attr("disabled", "disabled");
						}
					}
				else{
					$('input[id="update"]').prop('disabled', false);	
				}
				for(var i = 0; i < aa.length; i++) {
			    var obj = aa[i];
				//alert(JSON.stringify(obj));
			    var deptid=$('#grid').find("input[value='"+obj.deptCode+"']").attr('id');
			   // alert("dept=="+deptid);
			    var deptIndex =deptid.replace('dept',''); 
			   // alert(deptIndex);
			   
			    var branchid=$("div.set"+deptIndex).find("input[value='"+obj.branchCode+"']").attr('id');
			   // alert("branch=="+branchid);
			    var totalbranch=$("div.set"+deptIndex).find("input[id^='branch']").size();
			    
			    var branchIndex = branchid.replace('branch',''); 
			    //alert("totalbranch="+totalbranch);
			    branchIndex=branchIndex%totalbranch;
			    if(branchIndex==0)
			    	{branchIndex=totalbranch;}
			   // alert("positionset"+deptIndex+branchIndex);
			    var positionid=$("div.postionset"+deptIndex+branchIndex).find("input[value='"+obj.position+"']").attr('id');
			    //alert("posID=="+positionid);
			    var positionIndex = positionid.replace('position',''); 
			 
			    $("#empPrevYear"+positionIndex).val(obj.empPrevYear);
			    $("#empExisting"+positionIndex).val(obj.empExisting);
			   $("#empVacancy"+positionIndex).val(obj.empVacancy); 
			   
				}
								
				},
		  async:false
		});
	
	
	}

	
}
</script>
<script>
function onSaveGrid(){
	var i=0;
	var j=0;
	var deptSaveCount=arguments[0];//1
	var branchSavelength=arguments[1];//3
	var postionSavelength=arguments[2];//21
	var postionSaveCount=arguments[3];//7
	var deptCount=$("input[id^='dept']").size(); // to count the no of dept generated dynamically
	var positionCount=$("input[id^='position']").size();
	var saveLength=positionCount/deptCount;
	var companyCode="";
	var deptCode="";
	var branchCode="";
	var budgetFlowStatus="";
	var Periodcode="";
	var budgetType="";
	var budgetSubType="";
	var position="";
	var empPrevYear="";
	var empExisting="";
	var empVacancy="";
	var empProposed="";
	var empNewPosition="";
	var empApprovedPosition="";
	
	alert("positionlength=="+postionSavelength);
	var startIndex=(deptSaveCount*saveLength)-saveLength+1; // to calculate the startIndex of the grid
	alert("startIndex=="+startIndex);
	var endIndex=postionSavelength;
	alert("endIndex=="+endIndex);
	for(j=startIndex;j<=endIndex;j++)
	{
		empVacancy=$("#empVacancy"+j).val();
		if(null==empVacancy || empVacancy.trim()=="" || empVacancy === undefined)
			{
			budgetFlowStatus="STAGE0";
			break;
			}
		else{
			budgetFlowStatus="STAGE1";
		}
	}
	
	budgetType=$("#budgetType").val();
	budgetSubType=$("#budgetSubType").val();
	companyCode=$("#companyCode").val();
	Periodcode=$("#Periodcode").val();
	deptCode=$("#dept"+deptSaveCount).val();
	totalbranch=$("div.set"+deptSaveCount).find("input[id^='branch']").size();
	 jsonObj = [];
	// alert(branchSavelength);
	// alert(postionSaveCount);
	var startIndexBranch=branchSavelength-totalbranch+1;
	 var index=startIndex;
	for(i=startIndexBranch;i<=branchSavelength;i++)
		{
		branchCode=$("#branch"+i).val();
		//alert(branchCode);
		
			for(j=1;j<=postionSaveCount;j++)
			{
				
				position=$("#position"+index).val();
				
				empPrevYear=$("#empPrevYear"+index).val();
				empExisting=$("#empExisting"+index).val();
				empVacancy=$("#empVacancy"+index).val();
				
				
				 item = {}
			        item ["companyCode"] = companyCode;
			        item ["Periodcode"] = Periodcode;
			        item ["branchCode"] = branchCode;
			        item ["deptCode"] = deptCode;
			        item ["budgetFlowStatus"] = budgetFlowStatus;
			        item ["position"] = position;
			        item ["empPrevYear"] = empPrevYear;
			        item ["empExisting"] = empExisting;
			        item ["empVacancy"] = empVacancy;
			        item ["budgetType"] = budgetType;
			        item ["budgetSubType"] = budgetSubType;
			        
			        jsonObj.push(item);
			        index++;
				//alert(position+"==="+empPrevYear+"===="+empExisting+"====="+empVacancy+"=====");
				
				
				//alert("dept=="+dept);
			}
		}
	var jsonStr=  JSON.stringify(jsonObj);
var json1='{ "rows" : '+jsonStr+" }";
	// jsonStr= JSON.stringify(json1);
	//alert(jsonStr);
	var aa=confirm("Do you want to update changes?");
	if(aa)
	{
	 $.ajax({
         url: 'saveBudget.action',
         type: 'post',
         dataType: 'JSON',
         success: function (data) {
        	 
        	 var respMsg=data;
        	
        	if(respMsg== "Success"){
            alert("Records saved successfully!");}
        	else{
        		  alert(respMsg);
        	}
         },
         data: { 
             test: json1 // look here!
           },
     }).done(function() {
    	 for(j=1;j<=postionSavelength;j++)
			{
				empPrevYear=$("#empPrevYear"+j).val("");
				empExisting=$("#empExisting"+j).val("");
				empVacancy=$("#empVacancy"+j).val("");
			}
     });
	
	}
		
	}
function calVacancy()
{
	var vacancyLevel=arguments[0];
	var prevY = $("#empPrevYear"+vacancyLevel).val();
	var existing = $("#empExisting"+vacancyLevel).val();
	var vac=prevY-existing;
	if(vac<0)
		{
		vac=0;
		}
	$("#empVacancy"+vacancyLevel).val(vac);
	}
function delExisting(){
	var vacancyLevel=arguments[0];
	$("#empExisting"+vacancyLevel).val("");
	
}
</script>

<body>
<%			int i=0;
			int k=0;
			int deptCount=0;
			int branchCount=0;
			int positionCount=0;
			int positionsTotal=0;
%>
<% 
AccessObject accessObject = (AccessObject)request.getSession().getAttribute(OmniConstants.SESSION_PARAM_ACCESSOBJECT);
String mode = (String)request.getAttribute("mode");
String type = (String)request.getAttribute("type");
boolean flag = false;
if(null!=request.getAttribute("flag")){flag=(Boolean)request.getAttribute("flag"); }

mode = mode == null ? "" : mode;
type = type == null ? "" : type;
Map<String, String> deptCode=(Map<String, String>)request.getAttribute("deptCode");
if(null==deptCode){deptCode=new HashMap();}

Map<String, String> companyCode=(Map<String, String>)request.getAttribute("companyCode");
if(null==companyCode){companyCode=new HashMap();}

Map<String, String> branchCode=(Map<String, String>)request.getAttribute("branchCode");
if(null==branchCode){branchCode=new HashMap();}

Map<String, String> Periodcode=(Map<String, String>)request.getAttribute("Periodcode");
if(null==Periodcode){Periodcode=new HashMap();}

Map<String, String> position=(Map<String, String>)request.getAttribute("position");
if(null==position){position=new HashMap();}

Map<String, String> budgetType=(Map<String, String>)request.getAttribute("budgetType");
if(null==budgetType){budgetType=new HashMap();}

Map<String, String> budgetSubType=(Map<String, String>)request.getAttribute("budgetSubType");
if(null==budgetSubType){budgetSubType=new HashMap();}

BudgetWorkforceMst budgetWorkforceMst = (BudgetWorkforceMst)request.getAttribute(OmniConstants.REQUEST_INPUTOBJ);
if(null==budgetWorkforceMst){budgetWorkforceMst=new BudgetWorkforceMst();}
AppConfig appConfig =(AppConfig)accessObject.getAppConfig();
String dateFormat=appConfig.getDateFormatStr();
request.setAttribute("dateFormat", dateFormat);
String errMsg = (String)request.getAttribute("errMsg");
if(null==errMsg){errMsg="";}
%>

<div class="row container_bg">
<%if((mode.equals(OmniConstants.USERCREATEACTION) || mode.equalsIgnoreCase(OmniConstants.USERUPDATEACTION)) && type=="")%>

<!--* fields are mandatory -->
<div class="head_spc">
<%if(mode.equalsIgnoreCase("CREATE")){ %>
<s:text name="global.bcrumbs.budgetWorkforceMst.creation"></s:text>
<%}if(mode.equalsIgnoreCase("UPDATE") && type.equalsIgnoreCase("")){%>
<s:text name="global.bcrumbs.budgetWorkforceMst.updation"></s:text>
<%} if(mode.equalsIgnoreCase("AUTHVIEW") || type.equalsIgnoreCase("USERVIEW")){ %>
<s:text name="global.bcrumbs.budgetWorkforceMst.view"></s:text>
<% } %>
</div>
<form id="budgetWorkforceMstCRUDForm" name = "budgetWorkforceMstCRUDForm" method="POST" action="saveBudgetWorkforceMst">


<div id="result">
	<div class="col-md-offset-1">
	 <div align="left" id="idErrTbl" class="errorDiv">
	<p id="el" style="color: #FF0040" align="left"></p>
	<p id="error" style="color: #FF0040" align="left"></p>
	<p id="error1" style="color: #FF0040" align="left"></p>
	
	<font face="Calibri" size="3" color="#FF0000"><b>
			<span id="simulationInfos"><%= errMsg %></span></b>
	</font>
	</div> </div>

	<input type="hidden" id="" name="mode" value="<%= mode%>">
	<input type="hidden" id="errMsg" name="errMsg" value="<%=request.getAttribute("errMsg")%>">
	<input type="hidden" name="type" value="<%= type %>" id="type">
	<input type="hidden" name="id" value="<%=budgetWorkforceMst.getId()%>">
	<input type="hidden" name="todayDate" id="todayDate" value="<%= request.getSession().getAttribute("oprnDate") %>">

	<div class="table-responsive"> 
		<table class="table borderless"> 
		<tr>
				<td class="col-md-2"><s:text name="global.lbl.budgetType" />&nbsp;<span id="mandatory" >*</span></td>
					<td class="col-md-3">  
							<select id="budgetType" name="budgetType" class="col-md-8 input-sm" style="background-color: lightYellow;" >
									<option value="">Select</option>
									<% for(Map.Entry<String, String> entry : budgetType.entrySet()) { 
								if(null !=	budgetWorkforceMst.getBudgetType() &&
								budgetWorkforceMst.getBudgetType().equalsIgnoreCase(entry.getKey())) { 
							%>
								<option value="<%=entry.getKey()%>" SELECTED><%=entry.getKey()%>-<%=entry.getValue() %></option>
							<%		} else { %>
								<option value="<%=entry.getKey() %>"><%=entry.getKey()%>-<%=entry.getValue() %></option>
							<%		} 			
								} 
							%>
							</select>
					</td>
					
					<td class="col-md-2"><s:text name="global.lbl.budgetSubType" />&nbsp;<span id="mandatory" >*</span></td>
					<td class="col-md-3">  
							<select id="budgetSubType" name="budgetSubType" class="col-md-8 input-sm" style="background-color: lightYellow;" >
									<option value="">Select</option>
									<% for(Map.Entry<String, String> entry : budgetSubType.entrySet()) { 
								if(null !=	budgetWorkforceMst.getBudgetSubType()&&
								budgetWorkforceMst.getBudgetSubType().equalsIgnoreCase(entry.getKey())) { 
							%>
								<option value="<%=entry.getKey()%>" SELECTED><%=entry.getKey()%>-<%=entry.getValue() %></option>
							<%		} else { %>
								<option value="<%=entry.getKey() %>"><%=entry.getKey()%>-<%=entry.getValue() %></option>
							<%		} 			
								} 
							%>
							</select>
					</td>
		</tr>
			<tr>
			<td class="col-md-2"><s:text name="global.lbl.Periodcode" />&nbsp;<span id="mandatory" >*</span></td>
					<td class="col-md-3">  
							<select id="Periodcode" name="Periodcode" class="col-md-8 input-sm" style="background-color: lightYellow;" >
									<option value="">Select</option>
									<% for(Map.Entry<String, String> entry : Periodcode.entrySet()) { 
								if(null !=	budgetWorkforceMst.getPeriodcode() &&
								budgetWorkforceMst.getPeriodcode().equalsIgnoreCase(entry.getKey())) { 
							%>
								<option value="<%=entry.getKey()%>" SELECTED><%=entry.getValue() %></option>
							<%		} else { %>
								<option value="<%=entry.getKey() %>"><%=entry.getValue() %></option>
							<%		} 			
								} 
							%>
							</select>
					</td>
			
					<td class="col-md-2"><s:text name="global.lbl.companyCode" />&nbsp;<span id="mandatory" >*</span></td>
					<td class="col-md-3">  
							<select id="companyCode" name="companyCode" class="col-md-8 input-sm" style="background-color: lightYellow;" >
									<option value="">Select</option>
									<% for(Map.Entry<String, String> entry : companyCode.entrySet()) { 
								if(null !=	budgetWorkforceMst.getCompanyCode() &&
								budgetWorkforceMst.getCompanyCode().equalsIgnoreCase(entry.getKey())) { 
							%>
								<option value="<%=entry.getKey()%>" SELECTED><%=entry.getKey()%>-<%=entry.getValue() %></option>
							<%		} else { %>
								<option value="<%=entry.getKey() %>"><%=entry.getKey()%>-<%=entry.getValue() %></option>
							<%		} 			
								} 
							%>
							</select>
					</td>
					
					
					
					<td class="col-md-3">
					<input type="button" tabindex="5" name="View" class="btn btn-primary" value="<s:text name="global.btn.view"/>" onClick="javascript:onView();"> 
					</td>
				</tr>
			
			
			
			</table>
<div style="margin-left:10%" id="grid">
			
	<% for(Map.Entry<String, String> dept : deptCode.entrySet()) { 
		 i=0;
		 deptCount++;
	%>
	<div class="set<%=deptCount%>">
	<input type="hidden" id="dept<%=deptCount %>" value="<%=dept.getKey()%>">
				<b>Department~<%=dept.getValue() %>	</b>		
		
	<table border="1" style="width:90%" class="gridtable"  > 
		<tr class="a">
			
			<% for(Map.Entry<String, String> branch : branchCode.entrySet()) { 
				branchCount++;
			    %>	
		
		<td>
		<input type="hidden" id="branch<%=branchCount %>" value="<%=branch.getKey()%>">
		<b>	Branch~<%=branch.getValue() %></b>
		</td>
				<% i++; } %>
		</tr>
		<tr class="a">
		<%for(int j=1;j<=i;j++){ %>
			<td>
			<div class="postionset<%=deptCount%><%=j%>">
				<table style="border:3px solid #ddd;">
					<tr style="border:3px solid #ddd;"><!-- labels for inner table -->
				
						<td style="border:3px solid #ddd;">
						<b><s:text name="Position" /></b>
						</td>
						<td style="border:3px solid #ddd;">
						<b><s:text name="Previous Year" /></b>
						
						</td>
						<td style="border:3px solid #ddd;">
						<b><s:text name="Existing" /></b>
						
						</td>
						<td style="border:3px solid #ddd;">
						<b><s:text name="Vacancy" /></b>
						
						</td>
						<%-- <td>
						<b><s:text name="Proposed" /></b>
						
						</td>
						<td>
						<b><s:text name="New" /></b>
						
						</td>
						<td>
						<b><s:text name="Approved" /></b>
						
						</td> --%>
						
					</tr>
					<%
					positionCount=0;
					for(Map.Entry<String, String> pos : position.entrySet()) { 
						positionsTotal++;
						positionCount++;
					k++;
					%>
				
					<tr class="a" style="border:3px solid #ddd;" ><!-- Values for inner table -->
					
						<td style="border:3px solid #ddd;">
						<input type="hidden" id="position<%=k %>" value="<%=pos.getKey()%>">
						<%=pos.getValue() %>
						</td>
						
						<td style="border:3px solid #ddd;">
						<input type="number" min="0" id="empPrevYear<%=k %>" style="width:100%" onchange="javascript:delExisting(<%=k %>);">
						
						</td>
						<td style="border:3px solid #ddd;">
						<input type="number" min="0" id="empExisting<%=k %>" style="width:100%" onchange="javascript:calVacancy(<%=k %>);">
					
						</td>
						<td style="border:3px solid #ddd;">
						<input type="number" min="0" id="empVacancy<%=k %>" style="width:100%" readonly>
						
						</td>
						<%-- <td>
						<input type="number" min="0" id="empProposed<%=k %>" style="width:100%" readonly>
						
						</td>
						<td>
						<input type="number" min="0" id="empNewPosition<%=k %>" style="width:100%" readonly>
						
						</td>
						<td>
						<input type="number" min="0" id="empApprovedPosition<%=k %>" style="width:100%" readonly>
						
						</td> --%>
					</tr>
				<%} %>
				</table>
				</div>
			</td>
			<%} %>
		</tr>
	
	</table>
	</div>
	<br/>
	<input type="button" id="update" name="update" class="btn btn-primary"  value="<s:text name="Update" />" onclick="javascript:onSaveGrid(<%=deptCount %>,<%=branchCount %>,<%=k %>,<%=positionCount %>)" >&nbsp;
	<br/><br/>
<%} %>

	</div>		
			
			
		
	</div>
<hr/><br/>
</div>
</form>
<script type="text/javascript">
	new FormValidator('budgetWorkforceMstCRUDForm',
		[{
			name : 'companyCode',
			display : 'Company Code',
			actions : 'required'
		}, {
			name : 'deptCode',
			display : 'Dept Code',
			actions : 'required'
		},
		{
			name : 'budgetStage',
			display : 'Budget Stage',
			actions : 'required'
		},
		 {
			name : 'userCode',
			display : 'User Code',
			actions : 'required'
		},
		],
		function(errors, evt) {
			if (errors.length > 0) {
				var errorString = '';
				document.getElementById("idErrTbl").style.display = "inline";
				for (var i = 0, errorLength = errors.length; i < errorLength; i++) {
					errorString += errors[i].message + '<br />';
				}
				document.getElementById("el").innerHTML = errorString;
			}
		});
	</script>

		<script src="ngjs/jquery-ui.min.js"></script>
	<script src="ngjs/jqModal.js"></script>
	<script src="ngjs/grid.locale-en.js"></script>
	<script src="ngjs/jquery.searchFilter.js"></script>
	<script src="nqjs/grid.locale-bg.js"></script>
	<script src="ngjs/jquery.jqGrid.js"></script>
	<script src='newjs/css_live_reload_init.js'></script>
</div>
</body>
===============================================================================================================================
# The Java Logic to handle JSON resq and response of above JSP 
===============================================================================================================================
package com.infrasofttech.omning.action.fm;

import java.text.ParseException;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.HashSet;
import java.util.List;
import java.util.Map;
import java.util.Set;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.apache.log4j.Logger;
import org.apache.struts2.ServletActionContext;
import org.apache.struts2.interceptor.ServletRequestAware;
import org.json.JSONArray;
import org.json.JSONException;
import org.json.JSONObject;

import com.google.gson.Gson;
import com.infrasofttech.beans.OmniWebTO;
import com.infrasofttech.domain.entities.AccessObject;
import com.infrasofttech.domain.entities.MenuMst;
import com.infrasofttech.domain.entities.fm.BudgetSubTypeMst;
import com.infrasofttech.domain.entities.fm.BudgetTypeMst;
import com.infrasofttech.domain.entities.fm.BudgetWorkforceMst;
import com.infrasofttech.domain.entities.fm.COASegmentMap;
import com.infrasofttech.domain.entities.fm.PeriodMst;
import com.infrasofttech.omning.services.fm.BudgetSubTypeMstService;
import com.infrasofttech.omning.services.fm.BudgetTypeMstService;
import com.infrasofttech.omning.services.fm.BudgetWorkforceMstService;
import com.infrasofttech.omning.services.fm.COASegmentMapService;
import com.infrasofttech.omning.services.fm.LookupSubValueService;
import com.infrasofttech.omning.services.fm.PeriodMstService;
import com.infrasofttech.omning.utils.AppCachingServlet;
import com.infrasofttech.omning.utils.OmniNGServiceFactory;
import com.infrasofttech.omning.utils.SpringUtil;
import com.infrasofttech.utils.OmniConstants;
import com.opensymphony.xwork2.ActionContext;
import com.opensymphony.xwork2.ActionSupport;

public class BudgetWorkforceMstCRUDAction extends ActionSupport implements ServletRequestAware {

	private static final long serialVersionUID = -1L;
	private static final Logger logger = Logger.getLogger(BudgetWorkforceMstCRUDAction.class);
	private HttpServletRequest request =(HttpServletRequest) ActionContext.getContext().get(ServletActionContext.HTTP_REQUEST);//
	private BudgetWorkforceMst budgetWorkforceMst;
	private String mode = "";
	private String errMsg = "";
	private boolean flag = false;
	private MenuMst menuMst;
	private String actionCode = "";
	private Long refPKId = -1L;
	private boolean isActive;
	private OmniWebTO omniWebTO = new OmniWebTO();
	private Long id;
	HttpSession session = null;
	private BudgetSubTypeMstService budgetSubTypeMstService = (BudgetSubTypeMstService) SpringUtil
			.getSpringUtil().getService("budgetSubTypeMstService");
	private BudgetTypeMstService budgetTypeMstService = (BudgetTypeMstService) SpringUtil.getSpringUtil().getService("budgetTypeMstService");
	private  COASegmentMapService coaSegmentMapService = (COASegmentMapService) SpringUtil.getSpringUtil().getService("coaSegmentMapService");
	private LookupSubValueService lookupSubValueService = (LookupSubValueService) SpringUtil.getSpringUtil().getService("lookupSubValueService");
	Integer totalNumberOfPages = 1;
	Integer currentPageNumber = 1;
	Integer totalNumberOfRecords = 0; 

	
	public String getBudgetWorkforceMst()throws Exception {
		String retVal = OmniConstants.LOGIN;
		try {
			setLookupValues();
			if (null != request.getParameter("id") && null != request.getParameter("mkck")) {
				mode = OmniConstants.UPDATE;
				id = Long.parseLong(request.getParameter("id"));
				budgetWorkforceMst = (BudgetWorkforceMst) getBudgetWorkforceMstService().find(id);
				flag = request.getParameter("mkck").equalsIgnoreCase("CHECKER");
			} else {
				if (null != request.getParameter("id")) {
					id = Long.parseLong(request.getParameter("id"));
				}
				else{
					id = -1L;
				}
				BudgetWorkforceMst entity = (BudgetWorkforceMst) getBudgetWorkforceMstService().find(id);
				if (null == entity) {
					budgetWorkforceMst  =  new BudgetWorkforceMst();
					mode = OmniConstants.CREATE;//
				} else {
					budgetWorkforceMst  =  entity;
				}
			}
			request.setAttribute(OmniConstants.REQUEST_INPUTOBJ, budgetWorkforceMst);
			request.setAttribute("type", request.getParameter("type"));
			request.setAttribute("mode", mode);
			request.setAttribute("flag", flag);
			retVal = OmniConstants.SUCCESS;
		} catch (Exception e) {
			logger.error("", e);
			omniWebTO.setErrorCode(e.getMessage());
			retVal = OmniConstants.INPUT;
		} finally {
			omniWebTO.setActionCode(OmniConstants.VIEW);
			omniWebTO.setRefPKId(refPKId);
			request.setAttribute(OmniConstants.REQUEST_OMNIWEBTO, omniWebTO);
		}
		return retVal;
	}

	@SuppressWarnings("unchecked")
	private void setLookupValues(){
		List<COASegmentMap>  coaSegmentMaps=coaSegmentMapService.getCOASegmentByTenant(getAccessObject().getTenantId());
	   
		Map<String, String> companyCode = new HashMap<String, String>();
		Set<String> coaSegmentMapsSetForComp =  new HashSet<String>();
		for(COASegmentMap segs:coaSegmentMaps )
		{
			coaSegmentMapsSetForComp.add(segs.getSegCode1());
		}
		for(String companyCodes :coaSegmentMapsSetForComp ){
			companyCode.putAll(lookupSubValueService.getLookupSubValueMapBySubValueCode(getAccessObject().getTenantId(), companyCodes));
		}
		request.setAttribute("companyCode", companyCode);
		//Company Lookup Ends
		
		//budgetType
		List<BudgetTypeMst> budgetTypeMst = new ArrayList();
		Map<String, String> budgetTypeMap = new HashMap<String, String>();
		if (null != budgetTypeMstService) {
			budgetTypeMst = budgetTypeMstService
					.getBudgetTypeMstListByTenant(getAccessObject()
							.getTenantId(), Boolean.TRUE, OmniConstants.AUTH_AUTHORIZED);
					//.getBudgetAttributeMstByTenant(getAccessObject()
							
		}
		for (BudgetTypeMst budgetType : budgetTypeMst) {

			budgetTypeMap.put(budgetType.getBudgetCode(), budgetType.getBudgetDesc());
		}
		budgetTypeMap = budgetTypeMap == null ? new HashMap<String, String>()
				: budgetTypeMap;
		request.setAttribute("budgetType", budgetTypeMap);
		
		//budgetsubtype lookup
		
		List<BudgetSubTypeMst> budgetSubTypeMst = new ArrayList();
		Map<String, String> budgetSubTypeMap = new HashMap<String, String>();
		if (null != budgetSubTypeMstService) {
			budgetSubTypeMst = budgetSubTypeMstService
					.getBudgetSubTypeMstListByTenant(getAccessObject()
							.getTenantId(), Boolean.TRUE, OmniConstants.AUTH_AUTHORIZED);
							
		}
		for(BudgetSubTypeMst budgetSubtype : budgetSubTypeMst)
		{
			budgetSubTypeMap.put(budgetSubtype.getBudgetSubCode(),budgetSubtype.getSubBudgetDesc());
		}
		budgetSubTypeMap = budgetSubTypeMap == null ? new HashMap<String, String>()
				: budgetSubTypeMap;
		request.setAttribute("budgetSubType", budgetSubTypeMap);
		
		//Dept Lookup Starts
		Map<String, String> deptCode = new HashMap<String, String>();
		Set<String> coaSegmentMapsSetForDept =  new HashSet<String>();	
		for(COASegmentMap segs:coaSegmentMaps )
		{
			coaSegmentMapsSetForDept.add(segs.getSegCode3());
		}
		for(String companyCodes :coaSegmentMapsSetForDept ){
			deptCode.putAll(lookupSubValueService.getLookupSubValueMapBySubValueCode(getAccessObject().getTenantId(), companyCodes));
		}
		request.setAttribute("deptCode", deptCode);
		//Dept Lookup Ends
		
		//Branch Lookup Starts
				Map<String, String> branchCode = new HashMap<String, String>();
				Set<String> coaSegmentMapsSetForBranch =  new HashSet<String>();	
				for(COASegmentMap segs:coaSegmentMaps )
				{
					coaSegmentMapsSetForBranch.add(segs.getSegCode2());
				}
				for(String branchCodes :coaSegmentMapsSetForBranch ){
					branchCode.putAll(lookupSubValueService.getLookupSubValueMapBySubValueCode(getAccessObject().getTenantId(), branchCodes));
				}
				request.setAttribute("branchCode", branchCode);
		//branchCode Lookup Ends
				
				String pos= getAccessObject().getTenantId()+"-"+getAccessObject().getLanguageCode()+"-"+"1000";
				Map<String, String> position = AppCachingServlet.globalLookupCache.get(pos);
				request.setAttribute("position", position);
		//PeriodCode
				PeriodMstService periodMstService = (PeriodMstService) SpringUtil.getSpringUtil().getService("periodMstService");
				Map<String, String> Periodcode = new HashMap<String, String>();
				List<PeriodMst> periodMsts = periodMstService.getPeriodMstListByType(getAccessObject().getTenantId(),"FINANCIALYEAR", OmniConstants.AUTH_AUTHORIZED,true);
				for (PeriodMst period : periodMsts) {
					Periodcode.put(
							period.getCode(),
							period.getPeriodDesc() + "-"
									+ period.getCode());
				}
				request.setAttribute("Periodcode", Periodcode);
			}
	

	

		public void createAndSaveBudget() throws JSONException, ParseException{
			AccessObject accessObject = (AccessObject) request.getSession().getAttribute(OmniConstants.SESSION_PARAM_ACCESSOBJECT);
			String jsonRespMsg="";
				String jsonStr=request.getParameter("test");
				if(null==jsonStr || jsonStr.isEmpty())
					{jsonStr="";}
				JSONObject jsonObj =new JSONObject(jsonStr);
				  JSONArray rows = jsonObj.getJSONArray("rows");
				    for (int i = 0 ; i < rows.length(); i++) {
				    	BudgetWorkforceMst budgetWorkforceMst=new BudgetWorkforceMst();
				    	final JSONObject budget = rows.getJSONObject(i);
				    	budgetWorkforceMst.setTenantId(getAccessObject().getTenantId());
				    	budgetWorkforceMst.setCreatedDate(getAccessObject().getOprtnDate());
				    	budgetWorkforceMst.setCreatedBy(getAccessObject().getLoginId());
				    	budgetWorkforceMst.setCompanyCode(budget.getString("companyCode"));
				    	budgetWorkforceMst.setDeptCode(budget.getString("deptCode"));
				    	budgetWorkforceMst.setBranchCode(budget.getString("branchCode"));
				    	budgetWorkforceMst.setPeriodcode(budget.getString("Periodcode"));
				    	budgetWorkforceMst.setBudgetType(budget.getString("budgetType"));
				    	budgetWorkforceMst.setBudgetSubType(budget.getString("budgetSubType"));
				    	budgetWorkforceMst.setPosition(budget.getString("position"));
				    	budgetWorkforceMst.setBudgetFlowStatus(budget.getString("budgetFlowStatus"));
				    	budgetWorkforceMst.setEmpPrevYear(budget.getString("empPrevYear"));
				    	budgetWorkforceMst.setEmpExisting(budget.getString("empExisting"));
				    	budgetWorkforceMst.setEmpVacancy(budget.getString("empVacancy"));
				    	budgetWorkforceMst.setStatusUpdateDate(getAccessObject().getOprtnDate());
				    	
				    	 budgetWorkforceMst = (BudgetWorkforceMst) getBudgetWorkforceMstService().saveOrUpdate(
									getAccessObject().getLoginId(), budgetWorkforceMst);
				    	 if(null==budgetWorkforceMst)
				    	 {
				    		 jsonRespMsg=jsonRespMsg+"\n Details of Branch "+budget.getString("branchCode")+"and Position"+budget.getString("position")+" not saved! ";
				    	 }
				    	 else
				    	 {
				    		 jsonRespMsg="Success";
				    	 }
				    }
				   
				 
				    		
				String json = new Gson().toJson(jsonRespMsg);
				try {
					HttpServletResponse resp = ServletActionContext.getResponse();
					resp.getWriter().print(json);
				} catch (Exception e) {
					logger.error("", e);
				} 
			}
	public void getBudgetWorkforceMstAjaxList() throws Exception {
		 
		String retVal = "";
		List<BudgetWorkforceMst> budgetWorkforceMsts = new  ArrayList<BudgetWorkforceMst>();
		
		try{
	
			HttpServletResponse resp = ServletActionContext.getResponse();
			resp.setContentType("application/json; charset=utf-8");
			AccessObject accessObject = (AccessObject) request.getSession().getAttribute(OmniConstants.SESSION_PARAM_ACCESSOBJECT);
			String companyCode = request.getParameter("companyCode");
			String Periodcode = request.getParameter("Periodcode");
			String budgetSubType = request.getParameter("budgetSubType");
			String budgetType = request.getParameter("budgetType");
		
			//if(null==companyCode || null==deptCode || companyCode.isEmpty() || deptCode.isEmpty())
			
				budgetWorkforceMsts= getBudgetWorkforceMstService().getBudgetWorkforceMstForGrid(getAccessObject().getTenantId(),budgetType, budgetSubType, companyCode, Periodcode);
			
				String json = new Gson().toJson(budgetWorkforceMsts);
				try {
					resp.getWriter().print(json);
				} catch (Exception e) {
					logger.error("", e);
				}
			
		} catch (Exception e) {
			if(budgetWorkforceMsts==null)
			{
				logger.error("No such Company and department mapped for Workforce Budget in period", e);
			}
			else{
			logger.error("", e);
			}
			omniWebTO.setErrorCode(e.getMessage());
			retVal = OmniConstants.INPUT;
		} finally {
			omniWebTO.setActionCode(OmniConstants.AUTHORIZED_LIST);
			request.setAttribute(OmniConstants.REQUEST_OMNIWEBTO, omniWebTO);
		} 
	}		
	
	private List<BudgetWorkforceMst>  getBudgetWorkforceMstList(String tenantId, Boolean isActive) {
			return getBudgetWorkforceMstService().getBudgetWorkforceMstListByTenant(tenantId,
				isActive);
	}
	
	private BudgetWorkforceMstService  getBudgetWorkforceMstService(){
		return (BudgetWorkforceMstService) OmniNGServiceFactory
			.getService(OmniConstants.MENUCODE_BUDGETWORKFORCEMST);
	}

	private AccessObject getAccessObject() {
		return (AccessObject) request.getSession(false).getAttribute(
			OmniConstants.SESSION_PARAM_ACCESSOBJECT);
	}

	public void setServletRequest(HttpServletRequest arg0) {
		this.request = arg0;
	}


	private HttpServletResponse getResponse() {
		return ServletActionContext.getResponse();
	}
}

=======================================================================================================================================
# Image Zoom effect on the JFree chart or any image with jsp 
=======================================================================================================================================
<script>
  $(document).ready(function(){
    $('.img-zoom').hover(function() {
        $(this).addClass('transition');
 
    }, function() {
        $(this).removeClass('transition');
    });

    
  });
</script>
  <style>
  
.img-zoom {
    width: 260px;
    -webkit-transition: all .2s ease-in-out;
    -moz-transition: all .2s ease-in-out;
    -o-transition: all .2s ease-in-out;
    -ms-transition: all .2s ease-in-out;
      
}
 
.transition {
    -webkit-transform: scale(1.5); 
    -moz-transform: scale(1.5);
    -o-transform: scale(1.5);
    transform: scale(1.5);
   }
 </style>

	<div style="margin-top:30px;margin-left:20px" >
	<a download="PieBuckets.jpg" href="../omniNGLCS/drawChart_drawPIEChart.action?pieRule=BUCKETS">
	<img class="img-zoom" src="drawChart_drawPIEChart.action?pieRule=BUCKETS"/>
	</a>
	
	<a download="BarPTP.jpg" href="../omniNGLCS/drawChart_drawBARChart.action?barRule=PTP">
	<img class="img-zoom" src="drawChart_drawBARChart.action?barRule=PTP"/></a>
	
	<a download="BarOverdue.jpg" href="../omniNGLCS/drawChart_drawBARChart.action?barRule=OVERDUE">
	<img class="img-zoom" src="drawChart_drawBARChart.action?barRule=OVERDUE"/></a>
	
	<a download="BarLAWFIRMLAWER.jpg" href="../omniNGLCS/drawChart_drawBARChart.action?barRule=LAWFIRMLAWER">
	<img class="img-zoom" src="drawChart_drawBARChart.action?barRule=LAWFIRMLAWER"/></a>
	
	<a download="PieOverdueClosed.jpg" href="../omniNGLCS/drawChart_drawPIEChart.action?pieRule=OVERDUECLOSED">
	<img class="img-zoom" src="drawChart_drawPIEChart.action?pieRule=OVERDUECLOSED"/></a>
	
<a download="PieMESSAGEFREQ.jpg" href="../omniNGLCS/drawChart_drawPIEChart.action?pieRule=MESSAGEFREQ">
	<img class="img-zoom" src="drawChart_drawPIEChart.action?pieRule=MESSAGEFREQ"/></a>
	
	</div>

======================================================================================================================================================
# Load Lookup on change of combo with two at a time
======================================================================================================================================================
<script>
function loadActionTypeAndSubType(){
	 var templateCode = $("#templateCode").val();
	  var actionType = $("#actionType");
	  var actionSubType = $("#actionSubType");
	 var mode=$("#mode").val();
	  if( templateCode != '' || templateCode != null || templateCode != 'undefined' || templateCode!='Select') {
		
		$.getJSON('getMessageActionTypeAndSubtype.action',{
			templateCode : templateCode,
			mode:mode
			},function(jsonResponse) {
				var data1=jsonResponse[0], data2=jsonResponse[1]; 
				if(jsonResponse == "" ){				
					actionType.find('option').remove();
					actionSubType.find('option').remove();
					$('<option>').val('').text("Select...").appendTo(actionType);
					$('<option>').val('').text("Select...").appendTo(actionSubType);
					$('#error').html("<s:text name="global.error.gl.code.not.defined"/>");
					return;
				}else{
					actionType.find('option').remove();
					actionSubType.find('option').remove();
					
					$('<option>').val('').text("Select...").appendTo(actionType);
					$('<option>').val('').text("Select...").appendTo(actionSubType);
					$.each(data1, function(key, value) {					
						$('<option>').val(key).text(value).appendTo(actionType);
						$("#error").html("");
						});
					$.each(data2, function(key, value) {
						$('<option>').val(key).text(value).appendTo(actionSubType);
						$("#error").html("");
						});
				}
						
						
	          });


	  }
}
</script>
===================================================================================================================================================
# Notepad++ regex

values \(([0-9 ]*)\,  to   values \(

to_timestamp\(.*?\)  

a)
'([0-9]+)-+([0-9]+)-+([0-9]+) ([0-9]+):+([0-9]+):+([0-9]+)'
(for dates in YYYY-MM-DD mm:ss:SS)
b)
'([0-9]+)-+([0-9]+)-+([0-9]+) ([0-9]+):+([0-9]+):+([0-9]+).([0-9]+)'
(for dates in YYYY-MM-DD mm:ss:SS.zzz)
c)
'([0-9]+)-+([0-9]+)-+([0-9]+)'
(for dates in YYYY-MM-DD)

=====================================================================
