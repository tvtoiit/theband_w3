/**
* Copyright(c) Fujinet Co., Ltd.
* All rights reserved. 
* 
* @(#)SearchAction.java 01-00 2023/11/13
* 
* Version 1.00
* Last_Update 2023/11/13
*/
package fjs.cs.action;

import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.springframework.web.struts.ActionSupport;

import fjs.cs.common.Constants;
import fjs.cs.form.SearchForm;
import fjs.cs.model.MSTCUSTOMER;
import fjs.cs.model.MSTUSER;
import fjs.cs.service.SearchService;
/**
 * SearchAction
 * 
 * This class performs searching and paging
 * 
 * @version 1.00
 * @since 1.00
 * @author toi-tv
 */
public class SearchAction extends ActionSupport {
	
	/**
	 * Perform search and pagination
	 *
	 * @param mapping   An ActionMapping object that contains information about the mapping of the Action.
	 * @param form      An ActionForm object that holds the search request data.
	 * @param request   An HttpServletRequest object that contains information about the HTTP request.
	 * @param response  An HttpServletResponse object that contains information about the HTTP response.
	 * @return          An ActionForward object indicating the forward path for the request.
	 * @throws Exception    If any exception occurs during the execution of the method.
	 */
	@Override
	public ActionForward execute(ActionMapping mapping, ActionForm form, HttpServletRequest request,
			HttpServletResponse response) throws Exception {
		String forward = Constants.FORWARD_FAILURE;
		SearchForm searchForm = (SearchForm) form;
		String modeSearch = searchForm.getsMode();
		HttpSession session = request.getSession(true);
		
		List<MSTCUSTOMER> cus = null;
		String currentPageStr = null;
		int currentPage;
		int page = 0;
		 
		SearchService customerService = (SearchService) getWebApplicationContext().getBean(Constants.BEAN_SEARCH);
		MSTCUSTOMER customer = new MSTCUSTOMER();
		
		if(!Constants.MODE_SEARCH.equals(modeSearch)) {
			currentPageStr = searchForm.getCurrentPage();
		} 
		
		//Display userName
		MSTUSER userLoginSuccess = (MSTUSER) session.getAttribute("user");
		if (userLoginSuccess != null) {
			request.setAttribute("userLoginSuccess", userLoginSuccess.getUserName());	
		} else {
			return mapping.findForward(Constants.FORWARD_SUCCESS);
		}
		
		//if other formsearch is null then perform the job of saving the search value
		if (searchForm != null) {
			request.setAttribute("name", searchForm.getUserName());
			request.setAttribute("sex", searchForm.getSex());
			request.setAttribute("birthToDay", searchForm.getBrithTo());
			request.setAttribute("birthFromDay", searchForm.getBrithFrom());
		}
		
		//If there is a current page, take out page one
		if (currentPageStr != null && !currentPageStr.isEmpty()) {
			currentPage = Integer.parseInt(currentPageStr);
		} else {
			currentPage = Constants.PAGE_ONE;
		}
		
		//Get the total number of search results items
		int pageCount = totalPage(customerService, searchForm);
			
		page = getAction(currentPage, searchForm, pageCount);
		
		//Calculate the index position value to display the page number
		int indexPage = (page-1)*Constants.TOTAL_ITEM;
		searchForm.setIndex(indexPage);
		
		cus = handleSearch(searchForm, customerService, request);
		
		request.setAttribute("tag", page);
		customer.setPageData(cus);
		request.setAttribute("pageMax", pageCount);
		request.setAttribute("model", customer);

		//Display buttons as desired
		disableButtonsBasedOnPageCount(searchForm, customerService, cus, request, pageCount, page);
		
		return mapping.findForward(forward);
	}
	
	/**
     * Set flags for button disabling based on the pageCount.
     * 
     * @param request   HttpServletRequest object for setting attributes
     * @param pageCount Page count total
     */
    private void disableButtonsBasedOnPageCount(SearchForm searchForm, SearchService customerService, List<MSTCUSTOMER> cus, HttpServletRequest request, int pageCount , int page) {
    	int countCustomer = (int)customerService.countCustomerSearchResults(searchForm);
        if (pageCount == 0) {
            request.setAttribute("disableFirst", true);
            request.setAttribute("disablePrevious", true);
            request.setAttribute("disableNext", true);
            request.setAttribute("disableLast", true);
            request.setAttribute("disableDelete", true);
        } else if (pageCount > 0 && countCustomer <= Constants.TOTAL_ITEM) {
            request.setAttribute("disableFirst", true);
            request.setAttribute("disablePrevious", true);
            request.setAttribute("disableNext", true);
            request.setAttribute("disableLast", true);
        } else if (page == Constants.PAGE_ONE) {
            request.setAttribute("disableFirst", true);
            request.setAttribute("disablePrevious", true);
        } else if (page == pageCount) {
            request.setAttribute("disableNext", true);
            request.setAttribute("disableLast", true);
        }
    }
	
	/**
	 * Calculate total number of pages
	 * 
	 * @param customerService 	Call the get count page function
	 * @return pageCount 		Page count total
	 */
	private int totalPage(SearchService customerService, SearchForm searchForm) {
		int countCustomer = (int)customerService.countCustomerSearchResults(searchForm);
		int pageCount = (int)Math.ceil(countCustomer / Constants.TOTAL_ITEM);
		if (countCustomer % Constants.TOTAL_ITEM != 0) {
			pageCount++;
		}
		return pageCount;
	}

	/**
	 * Process searches based on user criteria
	 * 
	 * @param searchForm    Form value of search screen
	 * @param searchResult  A input list to search
	 * @param request       HttpServletRequest object for setting attributes
	 * @return              Returns a search list, or null if there are validation errors
	 */
	private List<MSTCUSTOMER> handleSearch(SearchForm searchForm, SearchService searchResult, HttpServletRequest request) {
	    // If there are no errors, perform a search and return results
	    List<MSTCUSTOMER> resultSearch = searchResult.getCustomerSearchResults(searchForm);
	    return resultSearch;
	}
	
	/**
	 * Pagination action
	 * 
	 * @param currentPage  Current page location
	 * @param searchForm   Search form data
	 * @param endPage	   Last page  
	 * @return 			   Returns the current page position
	 */
	 private int getAction(int currentPage,  SearchForm searchForm, int endPage) {
		 String mode = searchForm.getsMode();
	    	if (Constants.MODE_FIRST.equals(mode)) {
	    		return 1;
	    	} else if (Constants.MODE_PREVIOUS.equals(mode)) {
	            return Math.max(currentPage - 1, 1);
	        } else if (Constants.MODE_NEXT.equals(mode)) {
	            return Math.min(currentPage + 1, endPage);
	        } else if (Constants.MODE_LAST.equals(mode)) {
	            return endPage;
	        }
		    return currentPage;
	 }
	
}





















/**
* Copyright(c) Fujinet Co., Ltd.
* All rights reserved. 
* 
* @(#)EditAction.java 01-00 2023/11/14
* 
* Version 1.00
* Last_Update 2023/11/14
*/
package fjs.cs.action;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.apache.commons.beanutils.BeanUtils;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.springframework.web.struts.ActionSupport;

import fjs.cs.common.Constants;
import fjs.cs.form.EditForm;
import fjs.cs.model.MSTCUSTOMER;
import fjs.cs.model.MSTUSER;
import fjs.cs.service.EditService;
/**
 * EditAction
 * 
 * This function handles additions and corrections
 * 
 * @version 1.00
 * @since 1.00
 * @author toi-tv
 */
public class EditAction extends ActionSupport {
	/**
	 * Handles add and edit functions
	 *
	 * @param mapping   An ActionMapping object that contains information about the mapping of the Action.
	 * @param form      An ActionForm object contains data that requires editing
	 * @param request   An HttpServletRequest object that contains information about the HTTP request.
	 * @param response  An HttpServletResponse object that contains information about the HTTP response.
	 * @return          An ActionForward object indicating the forward path for the request.
	 * @throws Exception    If any exception occurs during the execution of the method.
	 */
	@Override
	public ActionForward execute(ActionMapping mapping, ActionForm form, HttpServletRequest request,
			HttpServletResponse response) throws Exception {
		String forward = Constants.FORWARD_FAILURE;
		EditForm editForm = (EditForm) form;
		HttpSession session = request.getSession(true);
		
		String editMode = editForm.getsMode();
		String customerId = editForm.getCustomerId();
		
		EditService editService = (EditService) getWebApplicationContext().getBean(Constants.BEAN_EDIT);
		
		MSTCUSTOMER customer = new MSTCUSTOMER();
		BeanUtils.copyProperties(customer, editForm);
		
		//Display userName
		MSTUSER userLoginSuccess = (MSTUSER) session.getAttribute("user");
		if (userLoginSuccess != null) {
			request.setAttribute("userLoginSuccess", userLoginSuccess.getUserName());	
		}
		
		/**
		 * If mode is save with empty customerId then save
		 * Additionally, if mode is save and customerId exists, perform edit
		 */
		if (Constants.MODE_SAVE.equals(editMode) && customerId == "") {
			setValueFormEdit(editForm, customer, userLoginSuccess);
			int loggedInPsnCd = userLoginSuccess.getPsnCd();
			customer.setInsertPSNCD(loggedInPsnCd);
			
			//Handle customer inserts
			editService.insertCustomer(customer);
			forward = Constants.FORWARD_SUCCESS;
		} else if (Constants.MODE_SAVE.equals(editMode) && customerId != null) {
			setValueFormEdit(editForm, customer, userLoginSuccess);
			
			//Handle customer update
			editService.updateCustomers(customer);
			forward = Constants.FORWARD_SUCCESS;
		}
		
		//Save the display value of the edit interface
		if (customerId != null && !Constants.MODE_SAVE.equals(editMode)) {
			customer = editService.getCustomerInByCustomer(customer);
			request.setAttribute("customerId", customer.getCustomerId());
			request.setAttribute("customerName", customer.getCustomerName());
			request.setAttribute("customerSex", customer.getSex());
			request.setAttribute("customerBirthDay", customer.getBirthDay());
			request.setAttribute("customerEmail", customer.getEmail());
			request.setAttribute("customerAddress", customer.getAddress());
		}
		return mapping.findForward(forward);
	}
	
	/**
	 * Set up value forms for customers
	 * 
	 * @param editForm 	Edit form
	 * @param customer	Customer entity
	 */
	private void setValueFormEdit(EditForm editForm, MSTCUSTOMER customer, MSTUSER userLoginSuccess) {
		int loggedInPsnCd = userLoginSuccess.getPsnCd();
		customer.setCustomerName(editForm.getCustomerName());
		customer.setSex(editForm.getSex());
		customer.setBirthDay(editForm.getBirthDay());
		customer.setEmail(editForm.getEmail());
		customer.setAddress(editForm.getAddress());
		customer.setUpdatePSNCD(loggedInPsnCd);
	}
}








/**
* Copyright(c) Fujinet Co., Ltd.
* All rights reserved. 
* 
* @(#)SearchForm.java 01-00 2023/11/13
* 
* Version 1.00
* Last_Update 2023/11/13
*/
package fjs.cs.form;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

import javax.servlet.http.HttpServletRequest;

import org.apache.struts.action.ActionErrors;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionMapping;
import org.apache.struts.action.ActionMessage;

import fjs.cs.common.Constants;

/**
 * SearchForm
 * 
 * @version 1.00
 * @since 1.00
 * @author toi-tv
 *
 */

public class SearchForm extends ActionForm {
	private static final long serialVersionUID = 1L;
	private String userName;
	private String sex;
	private String brithFrom;
	private String brithTo;
	private String[] selectedCustomers;
	private String currentPage;
	private int index = 0;
	private String sMode;
	
	/*Getter and Setter*/
	/**
	 * Returns the location of the current page
	 * 
	 * @return User's current page
	 */
	public String getCurrentPage() {
		return currentPage;
	}
	
	/**
	 * Set the user's current location
	 * 
	 * @param currentPage User's new site location
	 */
	public void setCurrentPage(String currentPage) {
		this.currentPage = currentPage;
	}
	
	/**
	 * Get the index position value of the page
	 * 
	 * @return Index position of the page
	 */
	public int getIndex() {
		return index;
	}
	
	/**
	 * Set the index position value
	 * 
	 * @param index The user's new index location
	 */
	public void setIndex(int index) {
		this.index = index;
	}
	
	/**
	 * Get selected customers
	 * 
	 * @return Customer value array is selected
	 */
	public String[] getSelectedCustomers() {
		return selectedCustomers;
	}
	
	/**
	 * Sets the value array value that the user selected
	 * 
	 * @param selectedCustomers Returns a new array of selected values
	 */
	public void setSelectedCustomers(String[] selectedCustomers) {
		this.selectedCustomers = selectedCustomers;
	}
	
	/**
	 * Get the value of the user's name
	 * 
	 * @return Returns the userName to retrieve
	 */
	public String getUserName() {
		return userName;
	}

	/**
	 * Set the user's name value
	 * 
	 * @param userName Pass in the userName with the value to be set
	 */
	public void setUserName(String userName) {
		this.userName = userName;
	}
	
	/**
	 * Get the user's gender
	 * 
	 * @return Gender of the user to get
	 */
	public String getSex() {
		return sex;
	}
	
	/**
	 * Set the user's gender value
	 * 
	 * @param sex Transmitting sex value needs to be established
	 */
	public void setSex(String sex) {
		this.sex = sex;
	}
	
	/**
	 * Get the value birthday from
	 * 
	 * @return The birthday from value needs to be obtained
	 */
	public String getBrithFrom() {
		return brithFrom;
	}
	
	/**
	 * Set the brithFrom value
	 * 
	 * @param brithFrom Pass in the new birthday form value
	 */
	public void setBrithFrom(String brithFrom) {
		this.brithFrom = brithFrom;
	}
	
	/**
	 * Get the value birthday to
	 * 
	 * @return Birthday value to get
	 */
	public String getBrithTo() {
		return brithTo;
	}
	
	/**
	 * Set the brithTo value
	 * 
	 * @param brithTo Pass in the new birthday to value
	 */
	public void setBrithTo(String brithTo) {
		this.brithTo = brithTo;
	}

	/**
	 * Get the mode value
	 * 
	 * @return Mode value just executed
	 */
	public String getsMode() {
		return sMode;
	}
	
	/**
	 * Set the mode value
	 * 
	 * @param sMode Pass in the mode value to set
	 */
	public void setsMode(String sMode) {
		this.sMode = sMode;
	}
	
	@Override
	public ActionErrors validate(ActionMapping mapping, HttpServletRequest request) {
		ActionErrors errors = new ActionErrors();
		
	    if (brithTo != null && !brithTo.isEmpty() && !isValidDateFormat(brithTo)) {
	        errors.add("brithTo", new ActionMessage("errors.userID"));
	    }

	    // Case 2: If birthdayFrom is not valid
	    if (brithFrom != null && !brithFrom.isEmpty() && !isValidDateFormat(brithFrom)) {
	        errors.add("brithFrom", new ActionMessage("errors.userID"));
	    }

	    // Case 3: If both birthdayFrom and birthdayTo are invalid or birthdayFrom is greater than birthdayTo
	    if ((brithFrom != null && brithFrom != ""  && !brithFrom.isEmpty() && !isStartDateBeforeEndDate(brithFrom, brithTo)) ||
	       (brithTo != null  && brithFrom != "" && !brithTo.isEmpty() && !isStartDateBeforeEndDate(brithFrom,brithTo))) {
	        errors.add("", new ActionMessage("errors.userID"));
	    }
	    request.setAttribute("errors", errors);
		return errors;
	}
	
	/**
	  * Function to check date format "YYYY/MM/DD"
	  * 
	  * @param date
	  * @return true if true
	  */
   private boolean isValidDateFormat(String date) {
       try {
       	// Use SimpleDateFormat to check the format
           SimpleDateFormat sdf = new SimpleDateFormat("yyyy/MM/dd");
           sdf.setLenient(false);
           sdf.parse(date);
           return true;
       } catch (ParseException e) {
           // If there is a ParseException error, the format is incorrect
           return false;
       }
   }
   
   /**
    * The function checks whether the start date is less than the end date
    * 
    * @param startDate Start day
    * @param endDate   End day
    * @return  Returns true if true
    */
   private boolean isStartDateBeforeEndDate(String startDate, String endDate) {
       try {
           SimpleDateFormat sdf = new SimpleDateFormat("yyyy/MM/dd");
           Date start = sdf.parse(startDate);
           Date end = sdf.parse(endDate);
           return start.before(end);
       } catch (ParseException e) {
           // If there is a ParseException error, it is considered invalid
           return false;
       }
   }
}
















"Trường hợp chưa đăng nhập từ màn hình Login, sử dụng url của màn hình Search thì vẫn có thể hiển thị màn hình.
Nếu đúng thì khi chưa đăng nhập từ màn hình Login, tiến hành dùng url của màn hình Search thì vẫn phải quay về màn hình Login."				




    










import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.util.Arrays;

public class FileUploadAction extends Action {

    @Override
    public ActionForward execute(ActionMapping mapping, ActionForm form, HttpServletRequest request, HttpServletResponse response) throws Exception {
        FileUploadForm fileUploadForm = (FileUploadForm) form;
        MultipartFile multipartFile = fileUploadForm.getFile();

        // Process and save the uploaded file
        String fileName = multipartFile.getFileName();
        String contentType = multipartFile.getContentType();
        long fileSize = multipartFile.getSize();
        InputStream inputStream = multipartFile.getInputStream();

        // Read the file into a list of lines
        InputStream inputStream = new FileInputStream(fileName);
        String[] lines = inputStream.readAllLines();

        // Do something with the lines
        for (String line : lines) {
            // Do something with the line
        }

        // Respond with a success message
        request.setAttribute("message", "File uploaded successfully.");
        return mapping.findForward("success");
    }
}







---------validate
@Override
	public ActionErrors validate(ActionMapping mapping, HttpServletRequest request) {
		ActionErrors errors = new ActionErrors();
		
		// Kiểm tra userId không được để trống
		if (userId == null || userId.trim().isEmpty()) {
	        errors.add("userId", new ActionMessage("error.required.userId"));
	    }
		
		// Kiểm tra passWord không được để trống
		if (passWord == null || passWord.trim().isEmpty()) {
	        errors.add("passWord", new ActionMessage("error.required.passWord"));
	    }
		request.setAttribute("errors", errors);
		return errors;
	}



-------------check
ActionErrors errors = new ActionErrors();
errors.add("login", new ActionMessage("message.error.user.not.exit"));








check data ---------------

package fjs.cs.action;

import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.util.ArrayList;
import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.struts.action.Action;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.apache.tomcat.util.http.fileupload.FileItemIterator;
import org.apache.tomcat.util.http.fileupload.FileItemStream;
import org.apache.tomcat.util.http.fileupload.disk.DiskFileItemFactory;
import org.apache.tomcat.util.http.fileupload.servlet.ServletFileUpload;

import fjs.cs.dao.impl.T004DaoImp;
import fjs.cs.dto.mstcustomer;

public class T004 extends Action {
	@Override
	public ActionForward execute(ActionMapping mapping, ActionForm form, HttpServletRequest request,
			HttpServletResponse response) throws Exception {
		mstcustomer t004Form = (mstcustomer)form;
		String action = t004Form.getAction();
		
		
		if ("import".equals(action)) {
			// Kiểm tra xem request có phải là multipart không
            boolean isMultipart = ServletFileUpload.isMultipartContent(request);
			T004DaoImp t004Dao = new T004DaoImp();
			if (isMultipart) {
                try {
                    // Tạo một factory để xử lý file upload
                    DiskFileItemFactory factory = new DiskFileItemFactory();

                    // Tạo một đối tượng upload handler
                    ServletFileUpload upload = new ServletFileUpload(factory);

                    // Phân tích request thành các items
                    FileItemIterator iter = upload.getItemIterator(request);

                    // Lặp qua các items và xử lý
                    while (iter.hasNext()) {
                        FileItemStream item = iter.next();
                        InputStream fileContent = item.openStream();

                        if (!item.isFormField()) {
                            BufferedReader reader = new BufferedReader(new InputStreamReader(fileContent));
                            String line;
                            int lineNumber = 1; // Dòng bắt đầu từ 1
                            List<mstcustomer> listResult = t004Dao.getAll();

                            while ((line = reader.readLine()) != null) {
                                // Xử lý từng dòng của file ở đây
                                String[] columns = line.split(",");

                                // Kiểm tra hợp lệ cho từng dòng
                                if (!isRowValid(columns[0], columns[1], listResult)) {
                                    System.out.println("Line " + lineNumber + " is invalid");
                                    // Hiển thị alert hoặc thực hiện xử lý khác tùy ý
                                }

                                lineNumber++;
                            }
                        }
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
	        
		}
		
		
		return mapping.findForward("false");
	}
	
	private boolean isRowValid(String customerId, String customerName, List<mstcustomer> listResult) {
        // Kiểm tra CUSTOMER_ID
        if (customerId != null && !customerId.trim().isEmpty()) {
            // Nếu CUSTOMER_ID không rỗng, kiểm tra xem có tồn tại trong listResult không
            if (!isCustomerIdValid(customerId, listResult)) {
                System.out.println("Line CUSTOMER_ID is null or doesn't exist");
                return false;
            }
        } else {
            // Nếu CUSTOMER_ID rỗng, thông báo lỗi
            System.out.println("Line CUSTOMER_ID is null or empty");
            return false;
        }

        // Kiểm tra CUSTOMER_NAME
        if (customerName == null || customerName.trim().isEmpty()) {
            // Nếu CUSTOMER_NAME rỗng, thông báo lỗi
            System.out.println("Line CUSTOMER_NAME is empty");
            return false;
        }

        // Nếu không có lỗi, trả về true
        return true;
    }
	
	private boolean isCustomerIdValid(String customerId, List<mstcustomer> listResult) {
        // Viết logic kiểm tra CUSTOMER_ID có tồn tại trong listResult không
        for (mstcustomer customer : listResult) {
            if (customerId.equals(customer.getCustomerId())) {
                return true;
            }
        }
        return false;
    }
}


------------------form 
<%@ page language="java" contentType="text/html; charset=UTF-8"
	pageEncoding="UTF-8"%>
<%@include file="../common/taglib.jsp" %>

<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Login - Training</title>
<style type="text/css">
	<%@include file="../WEB-INF/css/T001.css" %>
</style>
</head>
<style>
	.main-container {
    position: relative;
	}
	
	.customButton {
	    position: absolute;
	    top: 0;
	    right: 0;
	    padding: 10px 15px;
	    background-color: #4CAF50;
	    color: white;
	    cursor: pointer;
	    border: none;
	    border-radius: 5px;
	}

</style>
<body>
<%@include file="../common/web/header.jsp" %>

<form id="form-import" action="./import.do" method="POST" enctype="multipart/form-data" onsubmit="return handleImport()">
	<div class="main-container">
        <input type="text" id="importText" name="importText" readonly />
        <label for="fileInput" class="customButton">Browse</label>
        <input type="file" id="fileInput" style="display: none;" />
    </div>
	 <div class="btn-import">
         <button id="importButton" name="action" value="import">Import</button>
         <button type="button" id="cancelButton">Cancel</button>
     </div>
</form>
	
<%@include file="../common/web/footer.jsp" %>
<script>
	document.getElementById('fileInput').addEventListener('change', handleFileSelect);
	    
	function handleFileSelect() {
	    var fileInput = document.getElementById('fileInput');
	    var importText = document.getElementById('importText');

	    // Lấy tên file từ input file
	    var fileName = fileInput.files[0].name;

	    // Hiển thị tên file trong input text
	    importText.value = fileName;
	}
	function handleImport() {
	    var fileInput = document.getElementById('fileInput');
	    var importText = document.getElementById('importText');
	
	    // Kiểm tra nếu không có file import
	    if (fileInput.files.length === 0) {
	        alert("File import is not existed!");
	        return false;
	    }
	
	    // Kiểm tra phần mở rộng của file
	    var fileName = fileInput.files[0].name;
	    var fileExtension = fileName.split('.').pop().toLowerCase();
	    if (fileExtension !== 'csv') {
	        alert("File import is invalid");
	        return false;
	    }
	
	    // Kiểm tra kích thước hoặc nội dung của file import
	    if (fileInput.files[0].size === 0) {
	        alert("File import is empty");
	        return false;
	    }
	    
	    return true;
	}
</script>


</body>
</html>

------------------------------



public void handleExport(HttpServletRequest request, HttpServletResponse response, mstcustomer t002Form, T002DaoImp impT002) {
        try {
            String name = t002Form.getTxtCustomerName();
            String sex = t002Form.getSex();
            String birthdayFrom = t002Form.getTxtBirthdayFromName();
            String birthdayTo = t002Form.getTxtBirthdayToName();
            List<mstcustomer> dataToExport = impT002.getDataSearch(name, sex, birthdayFrom, birthdayTo);

            // Thiết lập các thông số cho HTTP response để trình duyệt hiểu được là file CSV
            response.setContentType("text/csv");
            response.setHeader("Content-Disposition", "attachment; filename=\"exported_data.csv\"");

            // Ghi dữ liệu ra OutputStream của response
            PrintWriter writer = response.getWriter();

            // Ghi dòng header của file CSV (tên cột) với định dạng bao bọc bởi dấu ngoặc kép
            writer.println("\"Customer Name\",\"Sex\",\"Birthday\"");

            // Ghi dữ liệu từ danh sách vào file CSV với định dạng bao bọc bởi dấu ngoặc kép
            for (mstcustomer customer : dataToExport) {
                writer.println("\"" + customer.getCustomerName() + "\",\"" + customer.getSex() + "\",\"" + customer.getBirthDay() + "\"");
            }

            // Đóng writer
            writer.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }






































<%@ page import="your.package.UserClass" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Display User Name</title>
</head>
<body>

<%
    // Retrieve the user object from the session
    UserClass user = (UserClass) session.getAttribute("user");

    // Check if the user object is not null
    if (user != null) {
        // Access the userName property of the user object
        String userName = user.getUserName();
%>
        <!-- Display the userName in the HTML -->
        <p>Welcome, <%= userName %>!</p>
<%
    } else {
%>
        <p>User not found in session</p>
<%
    }
%>

</body>
</html>









<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
"http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">
<hibernate-mapping>
    <class name="fjs.cs.model.MSTCUSTOMER" table="MSTCUSTOMER">
        <id name="customerId" type="int">
            <column name="CUSTOMER_ID" />
            <generator class="assigned" />
            <generator class="native"/>
        </id>
        <property name="customerName" type="string">
            <column name="CUSTOMER_NAME" length="50" />
        </property>
        <property name="sex" type="string">
            <column name="SEX" length="1" />
        </property>
        <property name="birthDay" type="string">
            <column name="BIRTHDAY" length="10" />
        </property>
        <property name="email" type="string">
            <column name="EMAIL" length="40" />
        </property>
        <property name="address" type="string">
            <column name="ADDRESS" length="256" />
        </property>
        <property name="deleteYMD" type="timestamp">
            <column name="DELETE_YMD" />
        </property>
        <property name="insertYMD" type="timestamp">
            <column name="INSERT_YMD" />
        </property>
        <property name="insertPSNCD" type="int">
            <column name="INSERT_PSN_CD" />
        </property>
        <property name="updateYMD" type="timestamp">
            <column name="UPDATE_YMD" />
        </property>
        <property name="updatePSNCD" type="int">
            <column name="UPDATE_PSN_CD" />
        </property>
    </class>
</hibernate-mapping>







BigDecimal loggedInPsnCd = ms.getPsnCd();
session.setAttribute("loggedInPsnCd", loggedInPsnCd);


// Lấy giá trị loggedInPsnCd từ session T001
HttpSession session = request.getSession();
BigDecimal loggedInPsnCd = (BigDecimal) session.getAttribute("loggedInPsnCd");




<!-- Customer.hbm.xml -->
<!DOCTYPE hibernate-mapping PUBLIC "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">
<hibernate-mapping package="your.package.containing.entities">

    <class name="Customer" table="MSTCUSTOMER">

        <id name="customerId" type="long">
            <column name="customer_id"/>
            <generator class="sequence">
                <param name="sequence">SEQ_CUSTOMER_ID</param>
            </generator>
        </id>

        <property name="customerName" type="string">
            <column name="customer_name"/>
        </property>

        <!-- Các trường khác -->

    </class>

</hibernate-mapping>



SỬ DỤNG 
public void addCustomer(Customer customer) {
        Session session = null;
        Transaction transaction = null;

        try {
            session = sessionFactory.openSession();
            transaction = session.beginTransaction();

            // Thêm customer vào cơ sở dữ liệu
            session.save(customer);

            transaction.commit();
        } catch (Exception e) {
            if (transaction != null) {
                transaction.rollback();
            }
            e.printStackTrace();
        } finally {
            if (session != null) {
                session.close();
            }
        }
    }












<logic:notEmpty name="check">
	<script>
    		alert("<bean:write name='check' />");
	</script>
</logic:notEmpty>

-----------------------------
<script>
    function validateForm() {
        var birthdayInput = document.getElementById('txtCustomerBirthday');
        var emailInput = document.getElementById('txtCustomerEmail');
        var errorMessageDiv = document.getElementById('error-message');

        // Kiểm tra định dạng birthday
        var birthdayRegex = /^\d{4}\/\d{2}\/\d{2}$/;
        if (!birthdayRegex.test(birthdayInput.value)) {
            errorMessageDiv.innerText = 'Không đúng định dạng ngày sinh (YYYY/MM/DD)';
            return false; // Ngừng xử lí và không submit form
        }

        // Kiểm tra định dạng email
        var emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        if (!emailRegex.test(emailInput.value)) {
            errorMessageDiv.innerText = 'Không đúng định dạng email';
            return false; // Ngừng xử lí và không submit form
        }

        // Nếu tất cả kiểm tra thành công, trả về true để submit form
        return true;
    }
</script>













 public void insertCustomer(MSTCUSTOMER customer) {
        Transaction tx = null;
        try {
            tx = getSession().beginTransaction();

            // Assuming MSTCUSTOMER class has corresponding properties and getters
            String hql = "INSERT INTO MSTCUSTOMER (CUSTOMER_NAME, SEX, BIRTHDAY, EMAIL, ADDRESS, DELETE_YMD, INSERT_YMD, INSERT_PSN_CD, UPDATE_YMD, UPDATE_PSN_CD) " +
                         "VALUES (:customerName, :sex, :birthday, :email, :address, NULL, CURRENT_TIMESTAMP, :insertPsnCd, CURRENT_TIMESTAMP, :updatePsnCd)";

            Query query = getSession().createQuery(hql);

            // Set parameters using named parameters
            query.setParameter("customerName", customer.getCustomerName());
            query.setParameter("sex", customer.getSex());
            query.setParameter("birthday", customer.getBirthday());
            query.setParameter("email", customer.getEmail());
            query.setParameter("address", customer.getAddress());
            query.setParameter("insertPsnCd", customer.getInsertPsnCd());
            query.setParameter("updatePsnCd", customer.getUpdatePsnCd());

            query.executeUpdate();
            tx.commit();
        } catch (Exception e) {
            if (tx != null) {
                tx.rollback();
            }
            e.printStackTrace();
        } 
    }



















public void updateCustomers(MSTCUSTOMER customer) {
    Transaction tx = null;
    try {
        tx = getSession().beginTransaction();

        String hql = "UPDATE MSTCUSTOMER SET CUSTOMER_NAME = ?, SEX = ?, BIRTHDAY = ?, EMAIL = ?, ADDRESS = ?, DELETE_YMD = NULL, UPDATE_YMD = CURRENT_TIMESTAMP WHERE CUSTOMER_ID = ?";
        Query query = getSession().createQuery(hql);

        // Set parameters using positional notation
        query.setParameter(0, customer.getCustomerName());
        query.setParameter(1, customer.getSex());
        query.setParameter(2, customer.getBirthday());
        query.setParameter(3, customer.getEmail());
        query.setParameter(4, customer.getAddress());
        query.setParameter(5, customer.getCustomerId());

        query.executeUpdate();
        tx.commit();
    } catch (Exception e) {
        if (tx != null) {
            tx.rollback();
        }
        e.printStackTrace();
    } 
}







public Long countCustomerSearchResults(SearchForm searchForm) {
    try {
        StringBuilder hql = new StringBuilder("SELECT COUNT(*) FROM MSTCUSTOMER WHERE DELETE_YMD IS NULL");

        if (searchForm.getUserName() != null && !searchForm.getUserName().isEmpty()) {
            hql.append(" AND CUSTOMER_NAME LIKE :name");
        }

        if (searchForm.getSex() != null && !searchForm.getSex().isEmpty()) {
            hql.append(" AND SEX = :sex");
        }

        if (searchForm.getBrithFrom() != null && !searchForm.getBrithFrom().isEmpty()) {
            hql.append(" AND BIRTHDAY >= :birthdayFrom");
        }

        if (searchForm.getBrithTo() != null && !searchForm.getBrithTo().isEmpty()) {
            hql.append(" AND BIRTHDAY <= :birthdayTo");
        }

        Query countQuery = getSession().createQuery(hql.toString());

        if (searchForm.getUserName() != null && !searchForm.getUserName().isEmpty()) {
            countQuery.setParameter("name", "%" + searchForm.getUserName() + "%");
        }

        if (searchForm.getSex() != null && !searchForm.getSex().isEmpty()) {
            countQuery.setParameter("sex", searchForm.getSex());
        }

        if (searchForm.getBrithFrom() != null && !searchForm.getBrithFrom().isEmpty()) {
            countQuery.setParameter("birthdayFrom", searchForm.getBrithFrom());
        }

        if (searchForm.getBrithTo() != null && !searchForm.getBrithTo().isEmpty()) {
            countQuery.setParameter("birthdayTo", searchForm.getBrithTo());
        }

        Long totalCount = (Long) countQuery.uniqueResult();
        return totalCount;
    } catch (Exception e) {
        e.printStackTrace();
        return null;
    }
}
















public List<MSTCUSTOMER> countCustomerSearchResults(SearchForm searchForm) {
	    try {
	        StringBuilder hql = new StringBuilder("SELECT COUNT(*) FROM MSTCUSTOMER WHERE DELETE_YMD IS NULL");
	        
	        //If the user searches by any condition, that condition will be appended
	        if (searchForm.getUserName() != null && !searchForm.getUserName().isEmpty()) {
	            hql.append(" AND CUSTOMER_NAME LIKE :name");
	        }

	        if (searchForm.getSex() != null && !searchForm.getSex().isEmpty()) {
	            hql.append(" AND SEX = :sex");
	        }

	        if (searchForm.getBrithFrom() != null && !searchForm.getBrithFrom().isEmpty()) {
	            hql.append(" AND BIRTHDAY >= :birthdayFrom");
	        }

	        if (searchForm.getBrithTo() != null && !searchForm.getBrithTo().isEmpty()) {
	            hql.append(" AND BIRTHDAY <= :birthdayTo");
	        }
	        
	        hql.append(" ORDER BY CUSTOMER_ID");

	        Query query = getSession().createQuery(hql.toString());

	        if (searchForm.getUserName() != null && !searchForm.getUserName().isEmpty()) {
	            query.setParameter("name", "%" + searchForm.getUserName() + "%");
	        }

	        if (searchForm.getSex() != null && !searchForm.getSex().isEmpty()) {
	            query.setParameter("sex", searchForm.getSex());
	        }

	        if (searchForm.getBrithFrom() != null && !searchForm.getBrithFrom().isEmpty()) {
	            query.setParameter("birthdayFrom", searchForm.getBrithFrom());
	        }

	        if (searchForm.getBrithTo() != null && !searchForm.getBrithTo().isEmpty()) {
	            query.setParameter("birthdayTo", searchForm.getBrithTo());
	        }
	        
	        //Paging conditions
	        query.setFirstResult(searchForm.getIndex());
	        query.setMaxResults(Constants.TOTAL_ITEM);
	        
	        @SuppressWarnings("unchecked")
			List<MSTCUSTOMER> customers = query.list();
	        return customers;
	    } catch (Exception e) {
	        e.printStackTrace();
	        return null;
	    }
	}












<input id="txtBirthdayForm" class ="input_Customer--common txtCustomerValidateFROM" name ="brithFrom" maxLength ="10" value="<logic:notEmpty name="birthFromDay"><bean:write name='birthFromDay'/></logic:notEmpty>"/>
<label for="html" class="handalSearch-customercommon handalSearch-BirthdayFrom__ngangcach">～</label>
<input id="txtBirthdayTo" class="input_Customer--common txtCustomerValidateTO" name ="brithTo" maxLength ="10" value="<logic:notEmpty name="birthToDay"><bean:write name='birthToDay'/></logic:notEmpty>"/>	





<form id="form-Search" action="./Search.do" method="POST">
			<div class="search-container__handalSearch">
				<div class="search-padding__handalSearch">
					<div class="search-container__handalSearch--margin handalSearch-customerName">
					<div class="handalSearch-customercommon handalSearch-customerName__text">Customer Name</div>
					<input id="txtCustomerName" class="input_Customer--common" name="userName" maxLength = "50" value="<logic:notEmpty name="name"><bean:write name='name'/></logic:notEmpty>"/>
				</div>
				<div class="search-container__handalSearch--margin handalSearch-customerSex">
					<div class="handalSearch-customercommon handalSearch-customerSex__text">Sex</div>
					<select name="sex" class="input_Customer--select" id="cboSex">
					    <option value=""></option>
					    <option value="0" <% if ("0".equals(request.getAttribute("sex"))) { %>selected<% } %>>Male</option>
					    <option value="1" <% if ("1".equals(request.getAttribute("sex"))) { %>selected<% } %>>Female</option>
					</select>
				</div>
					<div class="search-container__handalSearch--margin handalSearch-BirthdayFrom">
						<div class="handalSearch-customercommon handalSearch-BirthdayFrom__text">Birthday</div>
						<input id="txtBirthdayForm" class ="input_Customer--common txtCustomerValidateFROM" name ="brithFrom" maxLength ="10" value="<logic:notEmpty name="birthFromDay"><bean:write name='birthFromDay'/></logic:notEmpty>"/>
						<label for="html" class="handalSearch-customercommon handalSearch-BirthdayFrom__ngangcach">～</label>
						<input id="txtBirthdayTo" class="input_Customer--common txtCustomerValidateTO" name ="brithTo" maxLength ="10" value="<logic:notEmpty name="birthToDay"><bean:write name='birthToDay'/></logic:notEmpty>"/>	
					</div>
					<div class="handalSearch-btnSearch">
						<button type="submit" name="sMode" value="Search" id="btnSearch">Search</button>
					</div>
				</div>
			</div>
			<div class="search-container__btnContext--chuyenhuong">
			<input type="hidden" name="currentPage" value='<logic:notEmpty name="tag"><bean:write name="tag"/></logic:notEmpty>'/>
		    <div class="search-container__btnContext--start">
		    	<button name="sMode" <logic:notEmpty name="disableFirst">disabled</logic:notEmpty> class="search-btn search-btn__startend " value="first">&lt;&lt;</button>
	     		<button name="sMode" <logic:notEmpty name="disablePrevious">disabled</logic:notEmpty> class="search-btn search-btn__padding search-btn__soundstart" value="previous">&lt;</button>
		        <label for="html" class="search-container__btnContext--textstart">Previous</label>
		    </div>
		    <div class="search-container__btnContext--end">
		        <label for="html" class="search-container__btnContext--textend">Next</label>
		     	<button type="submit" <logic:notEmpty name="disableNext">disabled</logic:notEmpty> class="search-btn search-btn__padding" name="sMode" value="next">&gt;</button>  
		        <button type="submit" <logic:notEmpty name="disableLast">disabled</logic:notEmpty> class="search-btn search-btn__startend search-btn__soundend" name="sMode" value="last">&gt;&gt;</button>
		    </div>
		</div>
	</form>







<script type="text/javascript">
    function validateDelete() {
        // Lấy tất cả các checkbox với name="selectedCustomers"
        var checkboxes = document.getElementsByName("selectedCustomers");

        // Biến kiểm tra xem có ít nhất một checkbox được chọn hay không
        var atLeastOneChecked = false;

        // Duyệt qua tất cả các checkbox để kiểm tra
        for (var i = 0; i < checkboxes.length; i++) {
            if (checkboxes[i].checked) {
                atLeastOneChecked = true;
                break; // Nếu có ít nhất một checkbox được chọn, thoát khỏi vòng lặp
            }
        }

        // Kiểm tra giá trị của biến atLeastOneChecked
        if (!atLeastOneChecked) {
            // Hiển thị thông báo nếu không có checkbox nào được chọn
            alert("Please select at least one customer to delete.");
            return false; // Ngăn chặn việc submit form
        }

        // Nếu có ít nhất một checkbox được chọn, cho phép submit form
        return true;
    }
</script>



<button type="submit" name="deleteAction" value="delete" onclick="return validateDelete()" <logic:notEmpty name="disableDelete">disabled</logic:notEmpty> id="delete-btn" class="search-btn search-container__nav-btndelete">Delete </button>








<form id="form-Search" action="./Delete.do" method="POST">
		<div>
		<table class="search-container__table">
	        <tr class="search-container__table--tieude">
	        	<th><input type="checkbox" id="checkAll" name="checkboxAll" value="" onclick="toggleAllCheckboxes()"></th>
	            <th>Customer ID</th>
	            <th>Customer Name</th>
	            <th>Sex</th>
	            <th>Birthday</th>
	            <th>Address</th>
	        </tr>
		<logic:iterate id="dept" name="model" property="pageData">
				    <tr>
				        <td><input type="checkbox" name="selectedCustomers" value="<bean:write name='dept' property='customerId'/>"></td>
				        <td>
							<html:link action="/Edit">
							    <bean:write name="dept" property="customerId" />
							</html:link>
				        </td>
				        <td><bean:write name='dept' property='customerName' /></td>
				        <td><bean:write name='dept' property='sex' /></td>
				        <td><bean:write name='dept' property='birthDay' /></td>
				        <td><bean:write name='dept' property='address' /></td>
				    </tr>
				</logic:iterate>
	        
    	</table>
    	</div>
		<div class="search-container__btnnav">
			<a href="./save-user.do" class="search-container__nav-btnAdd">Add New</a>
			<button type="submit" name="deleteAction" value="delete" <logic:notEmpty name="disableDelete">disabled</logic:notEmpty> id="delete-btn" class="search-btn search-container__nav-btndelete">Delete </button>
		</div>
	</form>


<button type="submit" name="deleteAction" value="delete" <logic:notEmpty name="disableDelete">disabled</logic:notEmpty> id="delete-btn" class="search-btn search-container__nav-btndelete">Delete </button>


<logic:iterate id="dept" name="model" property="pageData">
    <tr>
	<td><input type="checkbox" name="selectedCustomers" value="<bean:write name='dept' property='customerId'/>"></td>
	<td>
			<html:link action="/Edit">
			    <bean:write name="dept" property="customerId" />
			</html:link>
	</td>
	<td><bean:write name='dept' property='customerName' /></td>
	<td><bean:write name='dept' property='sex' /></td>
	<td><bean:write name='dept' property='birthDay' /></td>
	<td><bean:write name='dept' property='address' /></td>
    </tr>
</logic:iterate>




<!-- Customer.hbm.xml -->
<hibernate-mapping>
    <class name="com.example.Customer" table="MSTCUSTOMER">
        <id name="customerId" type="java.lang.Long">
            <column name="CUSTOMER_ID" />
            <generator class="sequence">
                <param name="sequence">CUSTOMER_SEQ</param>
            </generator>
        </id>
        <property name="customerName" type="string">
            <column name="CUSTOMER_NAME" />
        </property>
        <!-- Các trường khác -->
    </class>
</hibernate-mapping>



import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;

public class CustomerDAO {
    private SessionFactory sessionFactory;

    // Constructor và getter/setter cho sessionFactory

    public void saveCustomer(Customer customer) {
        Session session = null;
        Transaction transaction = null;

        try {
            session = sessionFactory.openSession();
            transaction = session.beginTransaction();

            // Set INSERT_YMD và UPDATE_YMD sử dụng thời gian hiện tại
            customer.setInsertYmd(new Date());
            customer.setUpdateYmd(new Date());

            // Save the customer
            session.save(customer);
            
            transaction.commit();
        } catch (Exception e) {
            if (transaction != null) {
                transaction.rollback();
            }
            e.printStackTrace();
        } finally {
            if (session != null) {
                session.close();
            }
        }
    }
}








CREATE SEQUENCE your_sequence_name
  START WITH 1
  INCREMENT BY 1
  NO MINVALUE
  NO MAXVALUE
  NO CYCLE
  CACHE 1;









return new ArrayList<>();




------------Sử dụng
// get total item
int pageCount = totalPage(customerService);

// Set button disabling flags
disableButtonsBasedOnPageCount(request, pageCount);


/**
     * Set flags for button disabling based on the pageCount.
     * 
     * @param request   HttpServletRequest object for setting attributes
     * @param pageCount Page count total
     */
    private void disableButtonsBasedOnPageCount(HttpServletRequest request, int pageCount , int page) {
        boolean disableFirst = false;
        boolean disablePrevious = false;
        boolean disableNext = false;
        boolean disableLast = false;
        boolean disableDelete = false;

        if (pageCount == 0) {
            disableFirst = true;
            disablePrevious = true;
            disableNext = true;
            disableLast = true;
            disableDelete = true;
        } else if (pageCount > 0 && pageCount < Constants.TOTAL_ITEM) {
            disableFirst = true;
            disablePrevious = true;
            disableNext = true;
            disableLast = true;
        }

 	if (page == 1) {
            disableFirst = true;
            disablePrevious = true;
        }

        if (page == pageCount) {
            disableNext = true;
            disableLast = true;
        }

        // Set flags as request attributes
        request.setAttribute("disableFirst", disableFirst);
        request.setAttribute("disablePrevious", disablePrevious);
        request.setAttribute("disableNext", disableNext);
        request.setAttribute("disableLast", disableLast);
        request.setAttribute("disableDelete", disableDelete);
    }

}




















/**
* Copyright(c) Fujinet Co., Ltd.
* All rights reserved. 
* 
* @(#)SearchAction.java 01-00 2023/11/13
* 
* Version 1.00
* Last_Update 2023/11/13
*/
package fjs.cs.action;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.springframework.web.struts.ActionSupport;

import fjs.cs.common.Constants;
import fjs.cs.form.SearchForm;
import fjs.cs.model.MSTCUSTOMER;
import fjs.cs.service.SearchService;
/**
 * SearchAction
 * 
 * This class performs searching and paging
 * 
 * @version 1.00
 * @since 1.00
 * @author toi-tv
 */

public class SearchAction extends ActionSupport {
	
	/**
	 * Perform search and pagination
	 *
	 * @param mapping   An ActionMapping object that contains information about the mapping of the Action.
	 * @param form      An ActionForm object that holds the search request data.
	 * @param request   An HttpServletRequest object that contains information about the HTTP request.
	 * @param response  An HttpServletResponse object that contains information about the HTTP response.
	 * @return          An ActionForward object indicating the forward path for the request.
	 * @throws Exception    If any exception occurs during the execution of the method.
	 */
	@Override
	public ActionForward execute(ActionMapping mapping, ActionForm form, HttpServletRequest request,
			HttpServletResponse response) throws Exception {
		String forward = Constants.FORWARD_FAILURE;
		SearchForm searchForm = (SearchForm) form;
		String modeSearch = searchForm.getsMode();
		
		List<MSTCUSTOMER> cus = null;
		String currentPageStr = null;
		int page = 0;
		 
		SearchService customerService = (SearchService) getWebApplicationContext().getBean(Constants.BEAN_SEARCH);
		MSTCUSTOMER customer = new MSTCUSTOMER();
	   
		if(Constants.MODE_SEARCH.equals(modeSearch)) {
			request.setAttribute("name", searchForm.getUserName());
			request.setAttribute("sex", searchForm.getSex());
			request.setAttribute("birthToDay", searchForm.getBrithTo());
			request.setAttribute("birthFromDay", searchForm.getBrithFrom());

		}
		else {
			//lấy trang hiện tại 
			currentPageStr = searchForm.getCurrentPage();
		}
		
		//get Mode
		int currentPage;
		
		if (currentPageStr != null && !currentPageStr.isEmpty()) {
			currentPage = Integer.parseInt(currentPageStr);
		} else {
			currentPage = 1;
		}
		
		cus = handleSearch(searchForm, customerService, request);

		
		//get tottal item
		int pageCount = totalPage(customerService);
			
		page = getAction(currentPage, searchForm, pageCount);
		
		//Tính giá trị index
		int indexPage = (page*15)-15;
		searchForm.setIndex(indexPage);
		
		request.setAttribute("tag", page);
		customer.setPageData(cus);
		request.setAttribute("pageMax", pageCount);
		request.setAttribute("model", customer);

		
		disButton(cus, request, page, pageCount);
		
		return mapping.findForward(forward);
	}
	
	/**
	 * function handle button disabled
	 * 
	 * @param cus		List of search results 
	 * @param request	An HttpServletRequest object that contains information about the HTTP request.
	 * @param page		Current page number
	 * @param pageCount Total pages
	 */
	private void disButton(List<MSTCUSTOMER> cus, HttpServletRequest request, int page, int pageCount) {
		if (cus.size() <= 15) {
			request.setAttribute("disBtn", true);
		}
		
		if (cus.size() == 0) {
			request.setAttribute("disBtnDelete", true);
		}
		
		if (page == 1) {
			request.setAttribute("disBtnFirst", true);
		}
		
		if (page == pageCount) {
			request.setAttribute("disBtnLast", true);
		}
	}
	
	/**
	 * Calculate total number of pages
	 * 
	 * @param customerService 	Call the get count page function
	 * @return pageCount 		Page count total
	 */
	private int totalPage(SearchService customerService) {
		int countCustomer = customerService.getCountMstCustomer();
		int pageCount = (int)Math.ceil(countCustomer / Constants.TOTAL_ITEM);
		if (countCustomer % Constants.TOTAL_ITEM != 0) {
			pageCount++;
		}
		return pageCount;
	}

	/**
	 * Process searches based on user criteria
	 * 
	 * @param searchForm    Form value of search screen
	 * @param searchResult  A input list to search
	 * @param request       HttpServletRequest object for setting attributes
	 * @return              Returns a search list, or null if there are validation errors
	 */
	private List<MSTCUSTOMER> handleSearch(SearchForm searchForm, SearchService searchResult, HttpServletRequest request) {
	    String birthdayFrom = searchForm.getBrithFrom();
	    String birthdayTo = searchForm.getBrithTo();

	    // Trường hợp 1: Nếu birthdayTo không hợp lệ
	    if (birthdayTo != null && !birthdayTo.isEmpty() && !isValidDateFormat(birthdayTo)) {
	        request.setAttribute("invalidDateTo", true);
	        return null;
	    }

	    // Trường hợp 2: Nếu birthdayFrom không hợp lệ
	    if (birthdayFrom != null && !birthdayFrom.isEmpty() && !isValidDateFormat(birthdayFrom)) {
	        request.setAttribute("invalidDateFrom", true);
	        return null;
	    }

	    // Trường hợp 3: Nếu cả birthdayFrom và birthdayTo không hợp lệ hoặc birthdayFrom lớn hơn birthdayTo
	    if ((birthdayFrom != null && birthdayFrom != ""  && !birthdayFrom.isEmpty() && !isStartDateBeforeEndDate(birthdayFrom, birthdayTo)) ||
	       (birthdayTo != null  && birthdayFrom != "" && !birthdayTo.isEmpty() && !isStartDateBeforeEndDate(birthdayFrom,birthdayTo))) {
	        request.setAttribute("invalidDateRange", true);
	        return null;
	    }

	    // Nếu không có lỗi, thực hiện tìm kiếm và trả về kết quả
	    List<MSTCUSTOMER> resultSearch = searchResult.getCustomerSearchResults(searchForm);
	    return resultSearch;
	}
	
	/**
	 * Pagination action
	 * 
	 * @param currentPage  Current page location
	 * @param searchForm   Search form data
	 * @param endPage	   Last page  
	 * @return 			   Returns the current page position
	 */
	 private int getAction(int currentPage,  SearchForm searchForm, int endPage) {
		 String mode = searchForm.getsMode();
	    	if (Constants.MODE_FIRST.equals(mode)) {
	    		 return 1;
	    	} else if (Constants.MODE_PREVIOUS.equals(mode)) {
	            return Math.max(currentPage - 1, 1);
	        } else if (Constants.MODE_NEXT.equals(mode)) {
	            return Math.min(currentPage + 1, endPage);
	        } else if (Constants.MODE_LAST.equals(mode)) {
	            return endPage;
	        }
		    return currentPage;
	 }
	 
	 /**
	  * Function to check date format "YYYY/MM/DD"
	  * 
	  * @param date
	  * @return
	  */
    private boolean isValidDateFormat(String date) {
        try {
        	// Use SimpleDateFormat to check the format
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy/MM/dd");
            sdf.setLenient(false);
            sdf.parse(date);
            return true;
        } catch (ParseException e) {
            // If there is a ParseException error, the format is incorrect
            return false;
        }
    }
    
    
    /**
     * The function checks whether the start date is less than the end date
     * 
     * @param startDate Start day
     * @param endDate   End day
     * @return  Returns true if true
     */
    private boolean isStartDateBeforeEndDate(String startDate, String endDate) {
        try {
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy/MM/dd");
            Date start = sdf.parse(startDate);
            Date end = sdf.parse(endDate);
            return start.before(end);
        } catch (ParseException e) {
            // If there is a ParseException error, it is considered invalid
            return false;
        }
    }
	
}








<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
 <%@include file="../WEB-INF/common/Taglib.jsp" %>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title><bean:message key="search.title"/></title>
<style type="text/css">
	<%@include file="../WEB-INF/css/Base.css" %>
	<%@include file="../WEB-INF/css/Search.css" %>
</style>
<script src="./src/js/Search.js"></script>
</head>
<body>
<%@include file="../WEB-INF/common/Header.jsp" %>
	<div class="search-container">
		<div class="search-container__dan">
				<div class="search-container__text">
					Login > Search Customer
				</div>
			<div class="search-container__context">
				<div class="search-container__logo">
					<div>Welcome: <%=session.getAttribute("login")%> </div>
				</div>
				<a href="./Login.do" class="search-container__logout">Log Out</a>
			</div>
			<div class="search-container__line"></div>
		<form id="form-Search" action="./Search.do" method="POST">
			<div class="search-container__handalSearch">
				<div class="search-padding__handalSearch">
					<div class="search-container__handalSearch--margin handalSearch-customerName">
					<div class="handalSearch-customercommon handalSearch-customerName__text">Customer Name</div>
					<input id="txtCustomerName" class="input_Customer--common" name="userName" maxLength = "50" value="<logic:notEmpty name="name"><bean:write name='name'/></logic:notEmpty>"/>
				</div>
				<div class="search-container__handalSearch--margin handalSearch-customerSex">
					<div class="handalSearch-customercommon handalSearch-customerSex__text">Sex</div>
					<select name="sex" class="input_Customer--select" id="cboSex">
					    <option value="">blank</option>
					    <option value="0" <% if ("0".equals(request.getAttribute("sex"))) { %>selected<% } %>>Male</option>
					    <option value="1" <% if ("1".equals(request.getAttribute("sex"))) { %>selected<% } %>>Female</option>
					</select>
				</div>
					<div class="search-container__handalSearch--margin handalSearch-BirthdayFrom">
						<div class="handalSearch-customercommon handalSearch-BirthdayFrom__text">Birthday</div>
						<input id="txtBirthdayForm" class ="input_Customer--common txtCustomerValidateFROM" name ="brithFrom" maxLength ="10" value="<logic:notEmpty name="birthFromDay"><bean:write name='birthFromDay'/></logic:notEmpty>"/>
						<label for="html" class="handalSearch-customercommon handalSearch-BirthdayFrom__ngangcach">～</label>
						<input id="txtBirthdayTo" class="input_Customer--common txtCustomerValidateTO" name ="brithTo" maxLength ="10" value="<logic:notEmpty name="birthToDay"><bean:write name='birthToDay'/></logic:notEmpty>"/>	
					</div>
					<div class="handalSearch-btnSearch">
						<button type="submit" name="sMode" value="Search" id="btnSearch">Search</button>
					</div>
				</div>
			</div>
			<div class="search-container__btnContext--chuyenhuong">
			<input type="hidden" name="currentPage" value='<logic:notEmpty name="tag"><bean:write name="tag"/></logic:notEmpty>'/>
		    <div class="search-container__btnContext--start">
		    	<button name="sMode" <logic:notEmpty name="disBtn">disabled</logic:notEmpty> class="search-btn search-btn__startend " value="first">&lt;&lt;</button>
	     		<button name="sMode" <logic:notEmpty name="disBtn">disabled</logic:notEmpty> class="search-btn search-btn__padding search-btn__soundstart" value="previous">&lt;</button>
		        <label for="html" class="search-container__btnContext--textstart">Previous</label>
		    </div>
		    <div class="search-container__btnContext--end">
		        <label for="html" class="search-container__btnContext--textend">Next</label>
		     	<button type="submit" <logic:notEmpty name="disBtn">disabled</logic:notEmpty> class="search-btn search-btn__padding" name="sMode" value="next">&gt;</button>  
		        <button type="submit" <logic:notEmpty name="disBtn">disabled</logic:notEmpty> class="search-btn search-btn__startend search-btn__soundend" name="sMode" value="last">&gt;&gt;</button>
		    </div>
		</div>
	</form>
	<form id="form-Search" action="./Delete.do" method="POST">
		<div>
		<table class="search-container__table">
	        <tr class="search-container__table--tieude">
	        	<th><input type="checkbox" id="checkAll" name="checkboxAll" value="" onclick="toggleAllCheckboxes()"></th>
	            <th>Customer ID</th>
	            <th>Customer Name</th>
	            <th>Sex</th>
	            <th>Birthday</th>
	            <th>Address</th>
	        </tr>
		<logic:iterate id="dept" name="model" property="pageData">
				    <tr>
				        <td><input type="checkbox" name="selectedCustomers" value="<bean:write name='dept' property='customerId'/>"></td>
				        <td>
							<html:link action="/Edit">
							    <bean:write name="dept" property="customerId" />
							</html:link>
				        </td>
				        <td><bean:write name='dept' property='customerName' /></td>
				        <td><bean:write name='dept' property='sex' /></td>
				        <td><bean:write name='dept' property='birthDay' /></td>
				        <td><bean:write name='dept' property='address' /></td>
				    </tr>
				</logic:iterate>
	        
    	</table>
    	</div>
		<div class="search-container__btnnav">
			<a href="./save-user.do" class="search-container__nav-btnAdd">Add New</a>
			<button type="submit" <logic:notEmpty name="disBtnDelete">disabled</logic:notEmpty> id="delete-btn" class="search-container__nav-btndelete">Delete </button>
		</div>
	</form>
	</div>
</div>
<script>
<% 
	Boolean invalidDateFrom = (Boolean) request.getAttribute("invalidDateFrom");
	if (invalidDateFrom != null && invalidDateFrom) {
%>
	alert(" Invalid Birthday (From).");
<% } %>

<% 
	Boolean invalidDateTo = (Boolean) request.getAttribute("invalidDateTo");
	if (invalidDateTo != null && invalidDateTo) {
%>
	alert(" Invalid Birthday (To).");
<% } %>

<% 
	Boolean invalidDateRange = (Boolean) request.getAttribute("invalidDateRange");
	if (invalidDateRange != null && invalidDateRange) {
%>
	alert("There is an error in the range input of Birthday");
<% } %>

<% 
	Boolean checkDelete = (Boolean) request.getAttribute("recordCheckNull");
	if (checkDelete != null && checkDelete) {
%>
	alert("行を選択してください。");
<% } %>

</script>
<%@include file="../WEB-INF/common/Footer.jsp" %>	
</body>
</html>



















<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
 <%@include file="../WEB-INF/common/Taglib.jsp" %>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title><bean:message key="search.title"/></title>
<style type="text/css">
	<%@include file="../WEB-INF/css/Base.css" %>
	<%@include file="../WEB-INF/css/Search.css" %>
</style>
<script src="./src/js/Search.js"></script>
</head>
<body>
<%@include file="../WEB-INF/common/Header.jsp" %>
	<div class="search-container">
		<div class="search-container__dan">
				<div class="search-container__text">
					Login > Search Customer
				</div>
			<div class="search-container__context">
				<div class="search-container__logo">
					<div>Welcome: <%=session.getAttribute("login")%> </div>
				</div>
				<a href="./Login.do" class="search-container__logout">Log Out</a>
			</div>
			<div class="search-container__line"></div>
		<form id="form-Search" action="./Search.do" method="POST">
			<div class="search-container__handalSearch">
				<div class="search-padding__handalSearch">
					<div class="search-container__handalSearch--margin handalSearch-customerName">
					<div class="handalSearch-customercommon handalSearch-customerName__text">Customer Name</div>
					<input id="txtCustomerName" class="input_Customer--common" name="userName" maxLength = "50" value="<logic:notEmpty name="name"><bean:write name='name'/></logic:notEmpty>"/>
				</div>
				<div class="search-container__handalSearch--margin handalSearch-customerSex">
					<div class="handalSearch-customercommon handalSearch-customerSex__text">Sex</div>
					<select name="sex" class="input_Customer--select" id="cboSex">
					    <option value="">blank</option>
					    <option value="0" <% if ("0".equals(request.getAttribute("sex"))) { %>selected<% } %>>Male</option>
					    <option value="1" <% if ("1".equals(request.getAttribute("sex"))) { %>selected<% } %>>Female</option>
					</select>
				</div>
					<div class="search-container__handalSearch--margin handalSearch-BirthdayFrom">
						<div class="handalSearch-customercommon handalSearch-BirthdayFrom__text">Birthday</div>
						<input id="txtBirthdayForm" class ="input_Customer--common txtCustomerValidateFROM" name ="brithFrom" maxLength ="10" value="<logic:notEmpty name="birthFromDay"><bean:write name='birthFromDay'/></logic:notEmpty>"/>
						<label for="html" class="handalSearch-customercommon handalSearch-BirthdayFrom__ngangcach">～</label>
						<input id="txtBirthdayTo" class="input_Customer--common txtCustomerValidateTO" name ="brithTo" maxLength ="10" value="<logic:notEmpty name="birthToDay"><bean:write name='birthToDay'/></logic:notEmpty>"/>	
					</div>
					<div class="handalSearch-btnSearch">
						<button type="submit" name="sMode" value="Search" id="btnSearch">Search</button>
					</div>
				</div>
			</div>
			<div class="search-container__btnContext--chuyenhuong">
			<input type="hidden" name="currentPage" value='<logic:notEmpty name="tag"><bean:write name="tag"/></logic:notEmpty>'/>
		    <div class="search-container__btnContext--start">
		    	<button name="sMode" <logic:notEmpty name="disBtn">disabled</logic:notEmpty> class="search-btn search-btn__startend " value="first">&lt;&lt;</button>
	     		<button name="sMode" <logic:notEmpty name="disBtn">disabled</logic:notEmpty> class="search-btn search-btn__padding search-btn__soundstart" value="previous">&lt;</button>
		        <label for="html" class="search-container__btnContext--textstart">Previous</label>
		    </div>
		    <div class="search-container__btnContext--end">
		        <label for="html" class="search-container__btnContext--textend">Next</label>
		     	<button type="submit" <logic:notEmpty name="disBtn">disabled</logic:notEmpty> class="search-btn search-btn__padding" name="sMode" value="next">&gt;</button>  
		        <button type="submit" <logic:notEmpty name="disBtn">disabled</logic:notEmpty> class="search-btn search-btn__startend search-btn__soundend" name="sMode" value="last">&gt;&gt;</button>
		    </div>
		</div>
	</form>
	<form id="form-Search" action="./Delete.do" method="POST">
		<div>
		<table class="search-container__table">
	        <tr class="search-container__table--tieude">
	        	<th><input type="checkbox" id="checkAll" name="checkboxAll" value="" onclick="toggleAllCheckboxes()"></th>
	            <th>Customer ID</th>
	            <th>Customer Name</th>
	            <th>Sex</th>
	            <th>Birthday</th>
	            <th>Address</th>
	        </tr>
		<logic:iterate id="dept" name="model" property="pageData">
				    <tr>
				        <td><input type="checkbox" name="selectedCustomers" value="<bean:write name='dept' property='customerId'/>"></td>
				        <td>
							<html:link action="/Edit">
							    <bean:write name="dept" property="customerId" />
							</html:link>
				        </td>
				        <td><bean:write name='dept' property='customerName' /></td>
				        <td><bean:write name='dept' property='sex' /></td>
				        <td><bean:write name='dept' property='birthDay' /></td>
				        <td><bean:write name='dept' property='address' /></td>
				    </tr>
				</logic:iterate>
	        
    	</table>
    	</div>
		<div class="search-container__btnnav">
			<a href="./save-user.do" class="search-container__nav-btnAdd">Add New</a>
			<button type="submit" <logic:notEmpty name="disBtnDelete">disabled</logic:notEmpty> id="delete-btn" class="search-container__nav-btndelete">Delete </button>
		</div>
	</form>
	</div>
</div>
<script>
<% 
	Boolean invalidDateFrom = (Boolean) request.getAttribute("invalidDateFrom");
	if (invalidDateFrom != null && invalidDateFrom) {
%>
	alert(" Invalid Birthday (From).");
<% } %>

<% 
	Boolean invalidDateTo = (Boolean) request.getAttribute("invalidDateTo");
	if (invalidDateTo != null && invalidDateTo) {
%>
	alert(" Invalid Birthday (To).");
<% } %>

<% 
	Boolean invalidDateRange = (Boolean) request.getAttribute("invalidDateRange");
	if (invalidDateRange != null && invalidDateRange) {
%>
	alert("There is an error in the range input of Birthday");
<% } %>

<% 
	Boolean checkDelete = (Boolean) request.getAttribute("recordCheckNull");
	if (checkDelete != null && checkDelete) {
%>
	alert("行を選択してください。");
<% } %>

</script>
<%@include file="../WEB-INF/common/Footer.jsp" %>	
</body>
</html>









/**
* Copyright(c) Fujinet Co., Ltd.
* All rights reserved. 
* 
* @(#)SearchAction.java 01-00 2023/11/13
* 
* Version 1.00
* Last_Update 2023/11/13
*/
package fjs.cs.action;

import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.springframework.web.struts.ActionSupport;

import fjs.cs.common.Constants;
import fjs.cs.form.SearchForm;
import fjs.cs.model.MSTCUSTOMER;
import fjs.cs.service.SearchService;
/**
 * SearchAction
 * 
 * This class performs searching and paging
 * 
 * @version 1.00
 * @since 1.00
 * @author toi-tv
 */

public class SearchAction extends ActionSupport {
	
	/**
	 * Perform search and pagination
	 *
	 * @param mapping   An ActionMapping object that contains information about the mapping of the Action.
	 * @param form      An ActionForm object that holds the search request data.
	 * @param request   An HttpServletRequest object that contains information about the HTTP request.
	 * @param response  An HttpServletResponse object that contains information about the HTTP response.
	 * @return          An ActionForward object indicating the forward path for the request.
	 * @throws Exception    If any exception occurs during the execution of the method.
	 */
	@Override
	public ActionForward execute(ActionMapping mapping, ActionForm form, HttpServletRequest request,
			HttpServletResponse response) throws Exception {
		String forward = Constants.FORWARD_FAILURE;
		SearchForm searchForm = (SearchForm) form;
		String modeSearch = searchForm.getsMode();
		
		List<MSTCUSTOMER> cus = null;
		String currentPageStr = null;
		int page = 0;
		 
		SearchService customerService = (SearchService) getWebApplicationContext().getBean(Constants.BEAN_SEARCH);
		MSTCUSTOMER customer = new MSTCUSTOMER();
	   
		if(Constants.MODE_SEARCH.equals(modeSearch)) {
			request.setAttribute("name", searchForm.getUserName());
			request.setAttribute("sex", searchForm.getSex());
			request.setAttribute("birthToDay", searchForm.getBrithTo());
			request.setAttribute("birthFromDay", searchForm.getBrithFrom());

		}
		else {
			//lấy trang hiện tại 
			currentPageStr = searchForm.getCurrentPage();
		}
		
		//get Mode
		int currentPage;
		
		if (currentPageStr != null && !currentPageStr.isEmpty()) {
			currentPage = Integer.parseInt(currentPageStr);
		} else {
			currentPage = 1;
		}
		
		cus = handleSearch(searchForm, customerService, request);

		
		//get tottal item
		int pageCount = totalPage(customerService);
		

			
			page = getAction(currentPage, searchForm, pageCount);
			
			//Tính giá trị index
			int indexPage = (page*15)-15;
			searchForm.setIndex(indexPage);
			
			request.setAttribute("tag", page);
			customer.setPageData(cus);
			request.setAttribute("pageMax", pageCount);
			request.setAttribute("model", customer);

		
		disButton(cus, request, page, pageCount);
		
		return mapping.findForward(forward);
	}
	
	/**
	 * function handle button disabled
	 * 
	 * @param cus		List of search results 
	 * @param request	An HttpServletRequest object that contains information about the HTTP request.
	 * @param page		Current page number
	 * @param pageCount Total pages
	 */
	private void disButton(List<MSTCUSTOMER> cus, HttpServletRequest request, int page, int pageCount) {
		if (cus == null || cus.size() == 0) {
			request.setAttribute("disBtn", true);
		}
		
		if (cus == null || cus.size() <= 15) {
			request.setAttribute("disBtnDelete", true);
		}
		
		if (page == 1) {
			request.setAttribute("disBtnFirst", true);
		}
		
		if (page == pageCount) {
			request.setAttribute("disBtnLast", true);
		}
	}
	
	/**
	 * Calculate total number of pages
	 * 
	 * @param customerService 	Call the get count page function
	 * @return pageCount 		Page count total
	 */
	private int totalPage(SearchService customerService) {
		int countCustomer = customerService.getCountMstCustomer();
		int pageCount = (int)Math.ceil(countCustomer / Constants.TOTAL_ITEM);
		if (countCustomer % Constants.TOTAL_ITEM != 0) {
			pageCount++;
		}
		return pageCount;
	}

	/**
	 * Process searches based on user criteria
	 * 
	 * @param searchForm    Form value of search screen
	 * @param searchResult  A input list to search
	 * @param request       HttpServletRequest object for setting attributes
	 * @return              Returns a search list, or null if there are validation errors
	 */
	private List<MSTCUSTOMER> handleSearch(SearchForm searchForm, SearchService searchResult, HttpServletRequest request) {
	    String birthdayFrom = searchForm.getBrithFrom();
	    String birthdayTo = searchForm.getBrithTo();

	    // Trường hợp 1: Nếu birthdayTo không hợp lệ
	    if (birthdayTo != null && !birthdayTo.isEmpty() && !isValidDateFormat(birthdayTo)) {
	        request.setAttribute("invalidDateTo", true);
	        return null;
	    }

	    // Trường hợp 2: Nếu birthdayFrom không hợp lệ
	    if (birthdayFrom != null && !birthdayFrom.isEmpty() && !isValidDateFormat(birthdayFrom)) {
	        request.setAttribute("invalidDateFrom", true);
	        return null;
	    }

	    // Trường hợp 3: Nếu cả birthdayFrom và birthdayTo không hợp lệ hoặc birthdayFrom lớn hơn birthdayTo
	    if ((birthdayFrom != null && birthdayFrom != ""  && !birthdayFrom.isEmpty() && !isValidDateFormat(birthdayFrom)) ||
	       (birthdayTo != null  && birthdayFrom != "" && !birthdayTo.isEmpty() && !isValidDateFormat(birthdayTo))) {
	        request.setAttribute("invalidDateRange", true);
	        return null;
	    }

	    // Nếu không có lỗi, thực hiện tìm kiếm và trả về kết quả
	    List<MSTCUSTOMER> resultSearch = searchResult.getCustomerSearchResults(searchForm);
	    return resultSearch;
	}
	
	/**
	 * Pagination action
	 * 
	 * @param currentPage  Current page location
	 * @param searchForm   Search form data
	 * @param endPage	   Last page  
	 * @return 			   Returns the current page position
	 */
	 private int getAction(int currentPage,  SearchForm searchForm, int endPage) {
		 String mode = searchForm.getsMode();
	    	if (Constants.MODE_FIRST.equals(mode)) {
	    		 return 1;
	    	} else if (Constants.MODE_PREVIOUS.equals(mode)) {
	            return Math.max(currentPage - 1, 1);
	        } else if (Constants.MODE_NEXT.equals(mode)) {
	            return Math.min(currentPage + 1, endPage);
	        } else if (Constants.MODE_LAST.equals(mode)) {
	            return endPage;
	        }
		    return currentPage;
	 }
	 
	 /**
	  * Function to check date format "YYYY/MM/DD"
	  * 
	  * @param date
	  * @return
	  */
    private boolean isValidDateFormat(String date) {
        try {
        	// Use SimpleDateFormat to check the format
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy/MM/dd");
            sdf.setLenient(false);
            sdf.parse(date);
            return true;
        } catch (ParseException e) {
            // If there is a ParseException error, the format is incorrect
            return false;
        }
    }
    
    
    /**
     * The function checks whether the start date is less than the end date
     * 
     * @param startDate Start day
     * @param endDate   End day
     * @return  Returns true if true
     */
    private boolean isStartDateBeforeEndDate(String startDate, String endDate) {
        try {
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy/MM/dd");
            Date start = sdf.parse(startDate);
            Date end = sdf.parse(endDate);
            return start.before(end);
        } catch (ParseException e) {
            // If there is a ParseException error, it is considered invalid
            return false;
        }
    }
	
}










/**
	 * Process searches based on user criteria
	 * 
	 * @param searchForm 	Form value of search screen
	 * @param searchResult  A input list to search
	 * @return  			Returns a search list
	 */
	private List<MSTCUSTOMER> handleSearch(SearchForm searchForm, SearchService searchResult, HttpServletRequest request) {
		String birthdayTo = searchForm.getBrithTo();
		String birthdayFrom = searchForm.getBrithFrom();
		if (birthdayTo != "" && birthdayTo != null && !isValidDateFormat(birthdayTo)) {
            request.setAttribute("invalidDateTo", true);
            return null; 
        }
		
		if (birthdayFrom != "" && birthdayTo != null && !isValidDateFormat(birthdayFrom)) {
			request.setAttribute("invalidDateFrom", true);
			return null;
		}
		
		 if (birthdayFrom != "" && birthdayTo != null && birthdayFrom != null && birthdayFrom != null && !isStartDateBeforeEndDate(birthdayFrom, birthdayTo)) {
            // Nếu không, hiển thị thông báo và không thực hiện tìm kiếm
            request.setAttribute("invalidDateRange", true);
            return null; // Hoặc trả về null hoặc danh sách rỗng tùy thuộc vào yêu cầu của bạn
         }
		
		
		List<MSTCUSTOMER> resultSearch = searchResult.getCustomerSearchResults(searchForm);
        return resultSearch;
	 }

























public List<MSTCUSTOMER> getCustomerSearchResults(SearchForm searchForm) {
	    try {
	        StringBuilder hql = new StringBuilder("FROM MSTCUSTOMER WHERE DELETE_YMD IS NULL");

	        if (searchForm.getUserName() != null && !searchForm.getUserName().isEmpty()) {
	            hql.append(" AND CUSTOMER_NAME LIKE :name");
	        }

	        if (searchForm.getSex() != null && !searchForm.getSex().isEmpty()) {
	            hql.append(" AND SEX = :sex");
	        }

	        if (searchForm.getBrithFrom() != null && !searchForm.getBrithFrom().isEmpty()) {
	            hql.append(" AND BIRTHDAY >= :birthdayFrom");
	        }

	        if (searchForm.getBrithTo() != null && !searchForm.getBrithTo().isEmpty()) {
	            hql.append(" AND BIRTHDAY <= :birthdayTo");
	        }
	        hql.append(" ORDER BY CUSTOMER_ID");

	        Query query = getSession().createQuery(hql.toString());

	        if (searchForm.getUserName() != null && !searchForm.getUserName().isEmpty()) {
	            query.setParameter("name", "%" + searchForm.getUserName() + "%");
	        }

	        if (searchForm.getSex() != null && !searchForm.getSex().isEmpty()) {
	            query.setParameter("sex", searchForm.getSex());
	        }

	        if (searchForm.getBrithFrom() != null && !searchForm.getBrithFrom().isEmpty()) {
	            query.setParameter("birthdayFrom", searchForm.getBrithFrom());
	        }

	        if (searchForm.getBrithTo() != null && !searchForm.getBrithTo().isEmpty()) {
	            query.setParameter("birthdayTo", searchForm.getBrithTo());
	        }
	        query.setFirstResult(searchForm.getIndex());
	        query.setMaxResults(Constants.TOTAL_ITEM);
	        @SuppressWarnings("unchecked")
			List<MSTCUSTOMER> customers = query.list();
	        return customers;
	    } catch (Exception e) {
	        e.printStackTrace();
	        return null;
	    }
	}







public List<MSTCUSTOMER> getCustomerSearchResults(SearchForm searchForm) {
	    try {
	        StringBuilder hql = new StringBuilder("FROM MSTCUSTOMER WHERE DELETE_YMD IS NULL");

	        if (searchForm.getUserName() != null && !searchForm.getUserName().isEmpty()) {
	            hql.append(" AND CUSTOMER_NAME LIKE :name");
	        }

	        if (searchForm.getSex() != null && !searchForm.getSex().isEmpty()) {
	            hql.append(" AND SEX = :sex");
	        }

	        if (searchForm.getBrithFrom() != null && !searchForm.getBrithFrom().isEmpty()) {
	            hql.append(" AND BIRTHDAY >= :birthdayFrom");
	        }

	        if (searchForm.getBrithTo() != null && !searchForm.getBrithTo().isEmpty()) {
	            hql.append(" AND BIRTHDAY <= :birthdayTo");
	        }
	        hql.append(" ORDER BY CUSTOMER_ID");

	        Query query = getSession().createQuery(hql.toString());

	        if (searchForm.getUserName() != null && !searchForm.getUserName().isEmpty()) {
	            query.setParameter("name", "%" + searchForm.getUserName() + "%");
	        }

	        if (searchForm.getSex() != null && !searchForm.getSex().isEmpty()) {
	            query.setParameter("sex", searchForm.getSex());
	        }

	        if (searchForm.getBrithFrom() != null && !searchForm.getBrithFrom().isEmpty()) {
	            query.setParameter("birthdayFrom", searchForm.getBrithFrom());
	        }

	        if (searchForm.getBrithTo() != null && !searchForm.getBrithTo().isEmpty()) {
	            query.setParameter("birthdayTo", searchForm.getBrithTo());
	        }
	        query.setFirstResult(searchForm.getIndex());
	        query.setMaxResults(Constants.TOTAL_ITEM);
	        @SuppressWarnings("unchecked")
			List<?> customers = query.list();
	        
	        for (Iterator<?> iterator = customers.iterator(); iterator.hasNext();) {
	            MSTCUSTOMER customer = (MSTCUSTOMER) iterator.next();
	            customer.setSex("0".equals(customer.getSex()) ? "Male" : "Female");

	            customers.add(customer);
	        }
	        return customers;
	    } catch (Exception e) {
	        e.printStackTrace();
	        return null;
	    }
	}






---lưu giá trị giới tính
<select name="sex" class="input_Customer--select" id="cboSex">
    <option value="">blank</option>
    <option value="0" <% if ("0".equals(request.getAttribute("sex"))) { %>selected<% } %>>Male</option>
    <option value="1" <% if ("1".equals(request.getAttribute("sex"))) { %>selected<% } %>>Female</option>
</select>

----- CSS MÀN HÌNH SEARCH
.search-container {
	background-color: #ccffff;
}

.search-container__dan {
	padding: 20px;
}

.search-container__text {
	color: #000000;
}

.search-container__line {
	font-size: 10px;
	width: 100%;
	height: 10px;
	background-color: #3366ff;
	margin-top: 25px;
}

.search-container__context {
	padding-top: 20px;
	display: flex;
	justify-content: space-between;
}

.search-container__logo {
	justify-content: space-between;
}

.search-container__handalSearch {
	display: flex;
	margin-top: 30px;
	width: 100%;
	height: 60px;
	/*padding: 20px;*/
	background-color: #ffff99;
}

.handalSearch-customerName {
	display: flex;
	align-items: center;
}

.handalSearch-customerSex {
	display: flex;
	align-items: center;
}

.handalSearch-BirthdayFrom {
	display: flex;
	align-items: center;
}

.handalSearch-btnSearch {
	display: flex;
	align-items: center;
}

#btnSearch {
	height: 41px;
}

.handalSearch-customercommon {
	line-height: 41px;
}

.search-container__handalSearch--margin {
	flex: 1;
}

.handalSearch-customerName__text {
	min-width: 120px;
}

.handalSearch-customerSex__text {
	min-width: 30px;
}

.handalSearch-BirthdayFrom__text {
	min-width: 50px;
}

.search-container__btnContext--chuyenhuong {
	display: flex;
	justify-content: space-between;
}

.search-container__btnContext--start button {
	font-size: 20px;
	padding: 2px;
}

.search-container__btnContext--end button {
	font-size: 20px;
	padding: 2px;
}

.search-container__btnContext--start lable {
	font-size: 10px;
	margin-left: 15px;
}

.search-container__btnContext--end lable {
	font-size: 10px;
	margin-right: 15px;
}

.search-container__btnContext--chuyenhuong {
	margin: 30px 0;
}

.search-container__table {
	width: 100%;
}

table {
	border: 2px solid #339966;
	border-collapse: collapse;
	width: 100%;
}

table tr:nth-child(odd) {
	background-color: #ccffff;
}

table tr:nth-child(even) {
	background-color: #ffffff;
}

table tr:nth-child(1) {
	background-color: #339966;
}

th, td {
	text-align: left;
	padding: 10px;
}

.search-container__btnnav {
	display: flex;
	margin-top: 20px;
}

.search-container__btnnav .search-container__nav-btnAdd {
	margin-right: 10px;
}

.input_Customer--common {
	height: 35px;
}

.input_Customer--select {
	height: 40px;
}


----------  CHECKALL

<th><input type="checkbox" id="checkAll" name="checkboxAll" value="" onclick="toggleAllCheckboxes()"></th>


<script>
    // Hàm để xử lý sự kiện khi checkbox "Check All" được click
    function toggleAllCheckboxes() {
        var checkboxes = document.getElementsByName('selectedCustomers');
        var checkAllCheckbox = document.getElementById('checkAll');

        // Lấy giá trị checked của checkbox "Check All"
        var checkAllChecked = checkAllCheckbox.checked;

        // Duyệt qua tất cả các checkbox và cập nhật trạng thái checked của chúng
        for (var i = 0; i < checkboxes.length; i++) {
            checkboxes[i].checked = checkAllChecked;
        }
    }
</script>



-------- Disable button 
<button type="submit" <logic:notEmpty name="disBtnEndPage">disabled</logic:notEmpty>  name="pageAction" value="last">&gt;&gt;</button>

if (page == endPage) {
	request.setAttribute("disBtnEndPage", endPage);
}



















<input name="userId" id="txtUserID" value="<logic:notEmpty name='userId'><bean:write name='userId'/><logic:else/><%= "" %></logic:notEmpty>" maxlength="8" class="form-control" />




package com.demo.hibernate.dao;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.Iterator;
import java.util.List;

import org.hibernate.HibernateException;
import org.hibernate.Query;
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;
import org.springframework.orm.hibernate3.HibernateTemplate;
import org.springframework.orm.hibernate3.support.HibernateDaoSupport;

import com.demo.hibernate.entity.MSTCUSTOMER;

/**#################################################################
 * @author ANICET ERIC KOUAME
 * @Date: 20 mars 2017
 * @Description:
 *BookDao
 *#################################################################*/

public class BookDao extends HibernateDaoSupport {

	
	public List<MSTCUSTOMER> getAllCustomer() {
	    List<MSTCUSTOMER> customers = new ArrayList<>();

	    try {
	        Query query = getSession().createQuery("FROM MSTCUSTOMER WHERE DELETE_YMD IS NULL ORDER BY CUSTOMER_ID");
	        List<?> result = query.list();

	        for (Iterator<?> iterator = result.iterator(); iterator.hasNext();) {
	            MSTCUSTOMER customer = (MSTCUSTOMER) iterator.next();
	            // Thực hiện logic chuyển đổi giới tính ở đây nếu cần
	            // Ví dụ: customer.setSex(customer.getSex() == 0 ? "Male" : "Female");
	            //customer.setSex(Integer.toString(customer.getSex()).equals("0") ? "Male" : "Female");
	            customer.setSex("0".equals(customer.getSex()) ? "Male" : "Female");

	            customers.add(customer);
	        }
	    } catch (Exception e) {
	        e.printStackTrace();
	    }
	    return customers;
	}

	
	
	public List<MSTCUSTOMER> getCustomerSearchResults(String name, String sex, String birthdayFrom, String birthdayTo) {
	    try {
	        // Tạo câu truy vấn
	        StringBuilder hql = new StringBuilder("FROM MSTCUSTOMER WHERE DELETE_YMD IS NULL");

	        // Thêm điều kiện tìm kiếm theo tên nếu tên có
	        if (name != null && !name.isEmpty()) {
	            hql.append(" AND CUSTOMER_NAME LIKE :name");
	        }

	        // Thêm điều kiện tìm kiếm theo giới tính nếu giới tính có
	        if (sex != null && !sex.isEmpty()) {
	            hql.append(" AND SEX = :sex");
	        }

	        // Thêm điều kiện tìm kiếm theo ngày sinh bắt đầu nếu có
	        if (birthdayFrom != null && !birthdayFrom.isEmpty()) {
	            hql.append(" AND BIRTHDAY >= :birthdayFrom");
	        }

	        // Thêm điều kiện tìm kiếm theo ngày sinh kết thúc nếu có
	        if (birthdayTo != null && !birthdayTo.isEmpty()) {
	            hql.append(" AND BIRTHDAY <= :birthdayTo");
	        }

	        // Sắp xếp kết quả theo customerId
	        hql.append(" ORDER BY CUSTOMER_ID");

	        // Tạo Query và đặt tham số
	        Query query = getSession().createQuery(hql.toString());

	        // Đặt tham số
	        if (name != null && !name.isEmpty()) {
	            query.setParameter("name", "%" + name + "%");
	        }

	        if (sex != null && !sex.isEmpty()) {
	            query.setParameter("sex", sex);
	        }

	        if (birthdayFrom != null && !birthdayFrom.isEmpty()) {
	            query.setParameter("birthdayFrom", birthdayFrom);
	        }

	        if (birthdayTo != null && !birthdayTo.isEmpty()) {
	            query.setParameter("birthdayTo", birthdayTo);
	        }

	        // Thực hiện truy vấn
	        List<MSTCUSTOMER> customers = query.list();

	        return customers;
	    } catch (HibernateException e) {
	        e.printStackTrace();
	        return null;
	    }
	}
	
	public void deleteCustomers(String[] customerIds) {
	    Transaction tx = null;

	    try {
	        tx = getSession().beginTransaction();

	        String hql = "UPDATE MSTCUSTOMER SET DELETE_YMD = CURRENT_TIMESTAMP WHERE CUSTOMER_ID IN (:customerIds)";
	        Query query = getSession().createQuery(hql);

	        // Đặt danh sách giá trị vào tham số 'customerIds'
	        query.setParameterList("customerIds", Arrays.asList(customerIds));

	        int rowCount = query.executeUpdate();

	        System.out.println("Rows affected: " + rowCount);

	        tx.commit();
	    } catch (HibernateException e) {
	        if (tx != null) tx.rollback();
	        e.printStackTrace();
	    } 
	}
}


































import org.hibernate.Criteria;
import org.hibernate.Session;
import org.hibernate.Transaction;
import org.hibernate.criterion.Restrictions;

import java.util.ArrayList;
import java.util.List;

public class MstCustomerDAO {

    public List<MstCustomer> deleteData(String[] selecValue) {
        List<MstCustomer> listDelete = new ArrayList<>();

        Session session = HibernateUtil.getSessionFactory().openSession();
        Transaction tx = null;

        try {
            tx = session.beginTransaction();

            Criteria criteria = session.createCriteria(MstCustomer.class);
            criteria.add(Restrictions.in("customerId", Arrays.asList(selecValue)));

            // Lấy danh sách các khách hàng sẽ bị xóa
            listDelete = criteria.list();

            // Thực hiện cập nhật
            if (!listDelete.isEmpty()) {
                String hql = "UPDATE MstCustomer SET deleteYmd = CURRENT_TIMESTAMP WHERE customerId IN (:ids)";
                Query updateQuery = session.createQuery(hql);
                updateQuery.setParameterList("ids", Arrays.asList(selecValue));
                updateQuery.executeUpdate();
            }

            tx.commit();
        } catch (Exception e) {
            if (tx != null) {
                tx.rollback();
            }
            e.printStackTrace();
        } finally {
            session.close();
        }

        return listDelete;
    }
}









---------------


import org.hibernate.SQLQuery;
import org.hibernate.transform.Transformers;

public List<MSTCUSTOMER> getAllCustomer() {
    List<MSTCUSTOMER> currentCustomers = new ArrayList<>();
    try {
        Session session = getSession();
        String sql = "SELECT CUSTOMER_ID, CUSTOMER_NAME, CASE WHEN SEX = 0 THEN 'Male' ELSE 'Female' END AS SEX, BIRTHDAY, ADDRESS FROM " + Constants.TABLE_CUSTOMER + " WHERE DELETE_YMD IS NULL ORDER BY CUSTOMER_ID";
        
        SQLQuery getCustomerQuery = session.createSQLQuery(sql);
        getCustomerQuery.setResultTransformer(Transformers.aliasToBean(MSTCUSTOMER.class));
        currentCustomers = getCustomerQuery.list();
    } catch (Exception e) {
        e.printStackTrace();
    }

    return currentCustomers;
}






/**
* Copyright(c) Fujinet Co., Ltd.
* All rights reserved. 
* 
* @(#)MSTUSER.java 01-00 2023/06/20
* 
* Version 1.00
* Last_Update 2023/07/11
*/

package fjs.cs.model;

import java.io.Serializable;
import java.sql.Timestamp;

/**
 * MSTUSER
 * 
 * Data transfer object for MSTUSER entity.
 * 
 * @version 1.00
 * @since 1.00
 * 
 */

public class MSTUSER implements Serializable {

	private static final long serialVersionUID = 1L;
	
	private int psnCd;
	private String userID;
	private String password;
	private String userName;
	private Timestamp deleteYMD;
	private Timestamp insertYMD;
	private int insertPSNCD;
	private Timestamp updateYMD;
	private int updatePSNCD;

	/**
	 * Default constructor for MSTUSER class.
	 */
	public MSTUSER() {
		super();
	}
	
	/**
	 * Constructs a new instance of MSTUSER with the specified parameters.
	 *
	 * @param userID       the user ID
	 * @param password     the password
	 * @param userName     the user name
	 * @param deleteYMD    the deletion date
	 * @param insertYMD    the insertion date
	 * @param insertPSNCD  the insertion process code
	 * @param updateYMD    the update date
	 * @param updatePSNCD  the update process code
	 */
	
	public MSTUSER(String userID, String password, String userName, Timestamp deleteYMD, Timestamp insertYMD,
				   int insertPSNCD, Timestamp updateYMD, int updatePSNCD) {
		super();
		this.userID = userID;
		this.password = password;
		this.userName = userName;
		this.deleteYMD = deleteYMD;
		this.insertYMD = insertYMD;
		this.insertPSNCD = insertPSNCD;
		this.updateYMD = updateYMD;
		this.updatePSNCD = updatePSNCD;
	}

	/*Getter and Setter*/
	
	public int getPsnCd() {
		return psnCd;
	}

	public void setPsnCd(int psnCd) {
		this.psnCd = psnCd;
	}

	public String getUserID() {
		return userID;
	}

	public void setUserID(String userID) {
		this.userID = userID;
	}

	public String getPassword() {
		return password;
	}

	public void setPassword(String password) {
		this.password = password;
	}

	public String getUserName() {
		return userName;
	}

	public void setUserName(String userName) {
		this.userName = userName;
	}

	public Timestamp getDeleteYMD() {
		return deleteYMD;
	}

	public void setDeleteYMD(Timestamp deleteYMD) {
		this.deleteYMD = deleteYMD;
	}

	public Timestamp getInsertYMD() {
		return insertYMD;
	}

	public void setInsertYMD(Timestamp insertYMD) {
		this.insertYMD = insertYMD;
	}

	public int getInsertPSNCD() {
		return insertPSNCD;
	}

	public void setInsertPSNCD(int insertPSNCD) {
		this.insertPSNCD = insertPSNCD;
	}

	public Timestamp getUpdateYMD() {
		return updateYMD;
	}

	public void setUpdateYMD(Timestamp updateYMD) {
		this.updateYMD = updateYMD;
	}

	public int getUpdatePSNCD() {
		return updatePSNCD;
	}

	public void setUpdatePSNCD(int updatePSNCD) {
		this.updatePSNCD = updatePSNCD;
	}
}




















/**
* Copyright(c) Fujinet Co., Ltd.
* All rights reserved. 
* 
* @(#)LoginAction.java 01-00 2023/06/20
* 
* Version 1.00
* Last_Update 2023/07/11
*/

package fjs.cs.action;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.commons.beanutils.BeanUtils;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.apache.struts.action.ActionMessage;
import org.apache.struts.action.ActionMessages;
import org.springframework.web.struts.ActionSupport;

import fjs.cs.common.Constants;
import fjs.cs.form.LoginForm;
import fjs.cs.model.MSTUSER;
import fjs.cs.service.LoginService;

/**
 * LoginAction
 * 
 * This action handles the login function
 * 
 * @version 1.00
 * @since 1.00
 * 
 */

public class LoginAction extends ActionSupport {

	/**
	 * Processes the login request from the user.
	 *
	 * @param mapping   An ActionMapping object that contains information about the mapping of the Action.
	 * @param form      An ActionForm object that holds the login request data.
	 * @param request   An HttpServletRequest object that contains information about the HTTP request.
	 * @param response  An HttpServletResponse object that contains information about the HTTP response.
	 * @return          An ActionForward object indicating the forward path for the request.
	 * @throws Exception    If any exception occurs during the execution of the method.
	 */

	public ActionForward execute( ActionMapping mapping, ActionForm form, HttpServletRequest request,
								  HttpServletResponse response) throws Exception {
		String forward = Constants.FORWARD_FAILURE;
		LoginForm loginForm = (LoginForm) form;
		String userName = loginForm.getUsername();
		
		//get data username and password of url
		String passWord = loginForm.getPassword();
		String modeLogin = loginForm.getsMODE();
		
		ActionMessages errors = loginForm.validateForm(mapping, request);
		
		//Check the username and password, if they exist, then log in
		if (userName != null && passWord != null) {
			LoginService loginService = (LoginService) getWebApplicationContext().getBean(Constants.BEAN_LOGIN);
			MSTUSER user = new MSTUSER();
			loginForm.setUserID(userName);
			loginForm.setPassWord(passWord);
			BeanUtils.copyProperties(user, loginForm);
			MSTUSER loggedInUser = loginService.getLoggedInUser(user);

			if (loggedInUser != null) {
				return mapping.findForward(Constants.FORWARD_SUCCESS);
			} else {
				errors.add("", new ActionMessage("errors.notExist" + userName + passWord));
				this.saveErrors(request, errors);
			}
		}
		
		/**
		 * Check modelogin, if the user has not clicked on the Login button, then forward the login screen
		 */
		if (modeLogin == null || modeLogin.isEmpty()) {
			return mapping.findForward(forward);
		}
		
		//Check if the user clicks on the login button then logs in
		if (modeLogin.equals(Constants.MODE_LOGIN)) {
			if (!errors.isEmpty()) {
				this.saveErrors(request, errors);
			} else {
				LoginService loginService = (LoginService) getWebApplicationContext().getBean(Constants.BEAN_LOGIN);

				MSTUSER user = new MSTUSER();
				BeanUtils.copyProperties(user, loginForm);
				MSTUSER loggedInUser = loginService.getLoggedInUser(user);
				
				//	If loggedInUser is present, switch to the Search screen
				if (loggedInUser != null) {
					request.getSession().setAttribute("user", loggedInUser);
					forward = Constants.FORWARD_SUCCESS;
				} else {
					//Save login value when user is not in the database
					request.setAttribute("userId", loginForm.getUserID());
					request.setAttribute("passWord", loginForm.getPassWord());
					errors.add("", new ActionMessage("errors.notExist"));
					this.saveErrors(request, errors);
				}
			}
		}
		return mapping.findForward(forward);
	}
}















public ActionForward execute(ActionMapping mapping, ActionForm form, HttpServletRequest request, HttpServletResponse response) throws Exception {
    LoginForm loginForm = (LoginForm) form;
    String modeLogin = loginForm.getsMODE();

    if (loginFormHasValues(loginForm)) {
        return handleLogin(mapping, loginForm);
    } else if (isEmptyModeLogin(modeLogin)) {
        return mapping.findForward(Constants.FORWARD_FAILURE);
    } else if (modeLogin.equals(Constants.MODE_LOGIN)) {
        return handleLoginWithValidation(mapping, request, loginForm);
    }

    return mapping.findForward(Constants.FORWARD_FAILURE);
}

private boolean loginFormHasValues(LoginForm loginForm) {
    String userName = loginForm.getUsername();
    String passWord = loginForm.getPassword();
    return userName != null && passWord != null;
}

private boolean isEmptyModeLogin(String modeLogin) {
    return modeLogin == null || modeLogin.isEmpty();
}

private ActionForward handleLogin(ActionMapping mapping, LoginForm loginForm) {
    LoginService loginService = (LoginService) getWebApplicationContext().getBean(Constants.BEAN_LOGIN);
    MSTUSER user = new MSTUSER();
    loginForm.setUserID(loginForm.getUsername());
    loginForm.setPassWord(loginForm.getPassword());
    BeanUtils.copyProperties(user, loginForm);
    MSTUSER loggedInUser = loginService.getLoggedInUser(user);

    if (loggedInUser != null) {
        return mapping.findForward(Constants.FORWARD_SUCCESS);
    } else {
        ActionMessage errors = new ActionMessage("errors.urlLogin");
        return mapping.findForward(Constants.FORWARD_FAILURE);
    }
}

private ActionForward handleLoginWithValidation(ActionMapping mapping, HttpServletRequest request, LoginForm loginForm) {
    ActionMessages errors = loginForm.validateForm(mapping, request);

    if (!errors.isEmpty()) {
        this.saveErrors(request, errors);
    } else {
        LoginService loginService = (LoginService) getWebApplicationContext().getBean(Constants.BEAN_LOGIN);
        MSTUSER user = new MSTUSER();
        BeanUtils.copyProperties(user, loginForm);
        MSTUSER loggedInUser = loginService.getLoggedInUser(user);

        if (loggedInUser != null) {
            request.getSession().setAttribute("user", loggedInUser);
            return new ActionForward(Constants.FORWARD_SUCCESS);
        } else {
            request.setAttribute("userId", loginForm.getUserID());
            request.setAttribute("passWord", loginForm.getPassWord());
            errors.add("", new ActionMessage("errors.notExist"));
            this.saveErrors(request, errors);
        }
    }

    return mapping.findForward(Constants.FORWARD_FAILURE);
}




















public ActionForward execute( ActionMapping mapping, ActionForm form, HttpServletRequest request,
								  HttpServletResponse response) throws Exception {
		String forward = Constants.FORWARD_FAILURE;
		LoginForm loginForm = (LoginForm) form;
		String userName = loginForm.getUsername();
		String passWord = loginForm.getPassword();
		String modeLogin = loginForm.getsMODE();
		
		//System.out.println(userName + "-----" + passWord);
		
		if (userName != null && passWord != null) {
			LoginService loginService = (LoginService) getWebApplicationContext().getBean(Constants.BEAN_LOGIN);

			MSTUSER user = new MSTUSER();
			loginForm.setUserID(userName);
			loginForm.setPassWord(passWord);
			System.out.println(123);
			BeanUtils.copyProperties(user, loginForm);
			MSTUSER loggedInUser = loginService.getLoggedInUser(user);

			if (loggedInUser != null) {
				return mapping.findForward(Constants.FORWARD_SUCCESS);
			} else {
				ActionMessage errors = new ActionMessage("errors.urlLogin");
				return mapping.findForward(forward);
			}
		}
		System.out.println(modeLogin);
		if (modeLogin == null || modeLogin.isEmpty()) {
			return mapping.findForward(forward);
		} 
		
		if (modeLogin.equals(Constants.MODE_LOGIN)) {
			ActionMessages errors = loginForm.validateForm(mapping, request);
			if (!errors.isEmpty()) {
				this.saveErrors(request, errors);
			} else {
				LoginService loginService = (LoginService) getWebApplicationContext().getBean(Constants.BEAN_LOGIN);

				MSTUSER user = new MSTUSER();
				BeanUtils.copyProperties(user, loginForm);
				MSTUSER loggedInUser = loginService.getLoggedInUser(user);

				if (loggedInUser != null) {
					request.getSession().setAttribute("user", loggedInUser);
					forward = Constants.FORWARD_SUCCESS;
				} else {
					request.setAttribute("userId", loginForm.getUserID());
					request.setAttribute("passWord", loginForm.getPassWord());
					errors.add("", new ActionMessage("errors.notExist"));
					this.saveErrors(request, errors);
				}
			}
		}
		
		return mapping.findForward(forward);
	}









# theband_w3

1. Cơ sở dữ liệu:


CREATE DATABASE CUSTOME;
USE CUSTOME;

CREATE TABLE MSTUSER(
    PSN_CD NUMERIC(4) PRIMARY KEY NOT NULL,
    USERID VARCHAR(8),
    PASSWORD VARCHAR(8),
    USERNAME VARCHAR(40),
    DELETE_YMD DATETIME DEFAULT NULL,
    INSERT_YMD DATETIME,
    INSERT_PSN_CD NUMERIC(5) DEFAULT 0,
    UPDATE_YMD DATETIME DEFAULT NULL,
    UPDATE_PSN_CD NUMERIC(5) DEFAULT 0
);

CREATE TABLE MSTCUSTOMER(
    CUSTOMER_ID NUMERIC(8) PRIMARY KEY NOT NULL,
    CUSTOMER_NAME VARCHAR(50),
    SEX VARCHAR(1),
    BIRTHDAY VARCHAR(10),
    EMAIL VARCHAR(40),
    ADDRESS VARCHAR(256),
    DELETE_YMD DATETIME DEFAULT NULL,
    INSERT_YMD DATETIME,
    INSERT_PSN_CD NUMERIC(5) DEFAULT 0,
    UPDATE_YMD DATETIME DEFAULT NULL,
    UPDATE_PSN_CD NUMERIC(5) DEFAULT 0
);




package fjs.cs.model;

import java.math.BigDecimal;
import java.util.Date;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Table;
import javax.persistence.Temporal;
import javax.persistence.TemporalType;

@Entity
@Table(name = "MSTUSER")
public class MstUser {
    @Id
    @Column(name = "PSN_CD")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private BigDecimal psnCd;

    @Column(name = "USERID", length = 8)
    private String userId;

    @Column(name = "PASSWORD", length = 8)
    private String password;

    @Column(name = "USERNAME", length = 40)
    private String username;

    @Column(name = "DELETE_YMD")
    @Temporal(TemporalType.TIMESTAMP)
    private Date deleteYmd;

    @Column(name = "INSERT_YMD", updatable = false, insertable = false)
    @Temporal(TemporalType.TIMESTAMP)
    private Date insertYmd;

    @Column(name = "INSERT_PSN_CD", columnDefinition = "numeric(5,0)")
    private BigDecimal insertPsnCd;

    @Column(name = "UPDATE_YMD", updatable = false, insertable = false)
    @Temporal(TemporalType.TIMESTAMP)
    private Date updateYmd;

    @Column(name = "UPDATE_PSN_CD", columnDefinition = "numeric(5,0)")
    private BigDecimal updatePsnCd;

    // Constructors, getters, and setters
}



package fjs.cs.model;

import java.math.BigDecimal;
import java.sql.Timestamp;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Table;
import javax.persistence.Temporal;
import javax.persistence.TemporalType;

@Entity
@Table(name = "MSTCUSTOMER") // Tên bảng phải khớp với tên bảng trong cơ sở dữ liệu
public class MstCustomer {
    @Id
    @Column(name = "CUSTOMER_ID")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private BigDecimal customerId; // Sử dụng BigDecimal thay vì Long để phù hợp với SQL NUMERIC(8)

    @Column(name = "CUSTOMER_NAME", length = 50)
    private String customerName;

    @Column(name = "SEX", length = 1)
    private String sex;

    @Column(name = "BIRTHDAY", length = 10)
    private String birthDay;

    @Column(name = "EMAIL", length = 40)
    private String email;

    @Column(name = "ADDRESS", length = 256)
    private String address;

    @Column(name = "DELETE_YMD")
    @Temporal(TemporalType.TIMESTAMP)
    private Timestamp deleteYmd;

    @Column(name = "INSERT_YMD", updatable = false, insertable = false)
    @Temporal(TemporalType.TIMESTAMP)
    private Timestamp insertYmd;

    @Column(name = "INSERT_PSN_CD", columnDefinition = "numeric(5,0)")
    private BigDecimal insertPsnCd;

    @Column(name = "UPDATE_YMD", updatable = false, insertable = false)
    @Temporal(TemporalType.TIMESTAMP)
    private Timestamp updateYmd;

    @Column(name = "UPDATE_PSN_CD", columnDefinition = "numeric(5,0)")
    private BigDecimal updatePsnCd;

    // Constructors, getters, and setters
}






import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class AbstractCommon {
    public static Connection getConnection() {
        Connection conn = null;
        try {
            // Thay đổi class driver (ví dụ cho SQL Server)
            Class.forName("com.microsoft.sqlserver.jdbc.SQLServerDriver");

            // Thay đổi chuỗi kết nối (connection string) tương ứng
            String URL = "jdbc:sqlserver://localhost:1433;databaseName=yourDatabaseName";
            String USER = "yourUsername";
            String PASS = "yourPassword";
            conn = DriverManager.getConnection(URL, USER, PASS);
        } catch (SQLException | ClassNotFoundException e) {
            e.printStackTrace();
        }
        return conn;
    }
}





https://mkyong.com/spring/maven-spring-hibernate-mysql-example/



@Repository
public class LoginDao extends BaseDao {
	public List<Integer> checkLogin(MSTUSER userDto) {
		String sql = "SELECT COUNT(*) AS CNT FROM MSTUSER WHERE DELETE_YMD IS NULL AND USERID = ? AND PASSWORD = ?";
		List<Integer> list = _jdbcTemplate.query(sql);
		return list;
	}
}



package fjs.cs.dao;

import java.util.List;

import javax.transaction.Transaction;

import org.hibernate.classic.Session;
import org.springframework.stereotype.Repository;

import fjs.cs.model.MSTUSER;
@Repository
public class LoginDao extends BaseDao {
    public MSTUSER checkLogin(String userId, String password) {
        Session session = sessionFactory.openSession();
        Transaction tx = null;
        MSTUSER user = null;
        try {
            tx = session.beginTransaction();
            Criteria criteria = session.createCriteria(MSTUSER.class);
            criteria.add(Restrictions.eq("USERID", userId));
            criteria.add(Restrictions.eq("PASSWORD", password));
            criteria.add(Restrictions.isNull("DELETE_YMD"));
            user = (MSTUSER) criteria.uniqueResult();
            tx.commit();
        } catch (HibernateException e) {
            if (tx != null) {
                tx.rollback();
            }
            e.printStackTrace();
        } finally {
            session.close();
        }
        return user;
    }
}

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class HibernateTestConnection {

    public static void main(String[] args) {
        SessionFactory sessionFactory = null;
        Session session = null;

        try {
            // Tạo SessionFactory từ tệp cấu hình Hibernate (hibernate.cfg.xml)
            sessionFactory = new Configuration().configure("hibernate.cfg.xml").buildSessionFactory();

            // Mở một phiên làm việc Hibernate
            session = sessionFactory.openSession();

            // Kiểm tra kết nối đến cơ sở dữ liệu bằng cách thử lấy thông tin từ cơ sở dữ liệu
            String testQuery = "SELECT 1";
            int result = (int) session.createSQLQuery(testQuery).uniqueResult();
            System.out.println("Kết nối thành công! Kết quả truy vấn: " + result);
        } catch (Exception e) {
            e.printStackTrace();
            System.err.println("Kết nối thất bại: " + e.getMessage());
        } finally {
            // Đóng phiên làm việc
            if (session != null) {
                session.close();
            }
            // Đóng SessionFactory
            if (sessionFactory != null) {
                sessionFactory.close();
            }
        }
    }
}





------------------------

<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
"http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">
<hibernate-mapping>
	<class name="fjs.cs.model.MSTUSER" table="MSTUSER" >
		<id name="psnCd" type="int">
			<column name="PSN_CD"/>
			<generator class="assigned" />
		</id>
		<property name="userID" type="string">
			<column name="USERID" length="50"/>
		</property>
		<property name="password" type="string">
			<column name="PASSWORD" length="50"/>
		</property>
		<property name="userName" type="String">
			<column name="USERNAME" length="50"/>
		</property>
		<property name="deleteYMD" type="timestamp">
			<column name="DELETE_YMD"/>
		</property>
		<property name="insertYMD" type="timestamp">
			<column name="INSERT_YMD"/>
		</property>
		<property name="insertPSNCD" type="int">
			<column name="INSERT_PSN_CD"/>
		</property>
		<property name="updateYMD" type="timestamp">
			<column name="UPDATE_YMD"/>
		</property>
		<property name="updatePSNCD" type="int">
			<column name="UPDATE_PSN_CD"/>
		</property>
	</class>
</hibernate-mapping>




-------applicationContext----

<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:jdbc="http://www.springframework.org/schema/jdbc"
	xmlns:orm="http://www.springframework.org/schema/orm"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/jdbc/spring-jdbc.xsd
		http://www.springframework.org/schema/orm
		http://www.springframework.org/schema/orm/spring-orm.xsd">
	<!-- Configure Data Source -->
<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
	<property name="locations">
		<value>WEB-INF/src/app.properties</value>
	</property>
	<property name="ignoreResourceNotFound" value="true"/>
</bean>

<bean id="dataSource" 
         class="org.springframework.jdbc.datasource.DriverManagerDataSource">
	<property name="driverClassName" value="${jdbc.driverClassName}" />
	<property name="url" value="${jdbc.url}" />
	<property name="username" value="${jdbc.username}" />
	<property name="password" value="${jdbc.password}" />
</bean>

<bean id="sessionFactory" class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
	<property name="dataSource" ref="dataSource"/>
	<property name="mappingResources">
	<list>
		<value>fjs/cs/hibernate/MSTUSER.hbm.xml</value>
		<value>fjs/cs/hibernate/MSTCUSTOMER.hbm.xml</value>
	</list>
	</property>
	<property name="hibernateProperties">
		<props>
			<prop key="hibernate.dialect">org.hibernate.dialect.SQLServerDialect</prop>
			<prop key="hibernate.show_sql">true</prop>
			<prop key="hibernate.format_sql">true</prop>
		</props>
	</property>
</bean>

 <!-- Configure Struts 1  -->
<!-- DAO Beans  -->
	<bean id="loginDao" class="fjs.cs.dao.LoginDao">
		<property name="sessionFactory" ref="sessionFactory"></property>
	</bean>
	
	<bean id="customerDao" class="fjs.cs.dao.customerDao">
		<property name="sessionFactory" ref="sessionFactory"></property>
	</bean>
	
	<!-- Service Beans -->
	<bean id="loginService" class="fjs.cs.service.LoginService">
		<property name="loginDao" ref="loginDao"></property>
	</bean>
	<bean id="searchService" class="fjs.cs.service.SearchService">
		<property name="customerDao" ref="customerDao"></property>
	</bean>
	<bean id="deleteService" class="fjs.cs.service.DeleteService">
		<property name="customerDao" ref="customerDao"></property>
	</bean>
	<bean id="editService" class="fjs.cs.service.EditService">
		<property name="customerDao" ref="customerDao"></property>
	</bean>
	<bean id="importService" class="fjs.cs.service.ImportService">
		<property name="customerDao" ref="customerDao"></property>
	</bean>
	
</beans>



---------------------- app.properties

# -- set infomation for the database connection --
db.serverName=TOI-TV-VM\SQLEXPRESS
db.dbName=customersystem
db.portNumber=1433
db.userID=sa
db.password=Abc12345
db.driverClassName=com.microsoft.sqlserver.jdbc.SQLServerDriver
db.url=jdbc:sqlserver://localhost:1433;databaseName=customersystem



-----------------------web.xml

<?xml version="1.0" encoding="ISO-8859-1"?>

<!DOCTYPE web-app
  PUBLIC "-//Sun Microsystems, Inc.//DTD Web Application 2.2//EN"
  "http://java.sun.com/j2ee/dtds/web-app_2_2.dtd">

<web-app>
  <display-name>Struts Blank Application</display-name>

  <!-- Standard Action Servlet Configuration (with debugging) -->
  <servlet>
    <servlet-name>action</servlet-name>
    <servlet-class>org.apache.struts.action.ActionServlet</servlet-class>
    <init-param>
      <param-name>config</param-name>
      <param-value>/WEB-INF/struts-config.xml</param-value>
    </init-param>
    <init-param>
      <param-name>debug</param-name>
      <param-value>2</param-value>
    </init-param>
    <init-param>
      <param-name>detail</param-name>
      <param-value>2</param-value>
    </init-param>
    <load-on-startup>2</load-on-startup>
  </servlet>


  <!-- Standard Action Servlet Mapping -->
  <servlet-mapping>
    <servlet-name>action</servlet-name>
    <url-pattern>*.do</url-pattern>
  </servlet-mapping>
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>/WEB-INF/applicationContext.xml</param-value>
	</context-param>

  <!-- The Usual Welcome File List -->
  <welcome-file-list>
    <welcome-file>index.jsp</welcome-file>
  </welcome-file-list>


  <!-- Struts Tag Library Descriptors -->
  <taglib>
    <taglib-uri>/tags/struts-bean</taglib-uri>
    <taglib-location>/WEB-INF/struts-bean.tld</taglib-location>
  </taglib>

  <taglib>
    <taglib-uri>/tags/struts-html</taglib-uri>
    <taglib-location>/WEB-INF/struts-html.tld</taglib-location>
  </taglib>

  <taglib>
    <taglib-uri>/tags/struts-logic</taglib-uri>
    <taglib-location>/WEB-INF/struts-logic.tld</taglib-location>
  </taglib>

  <taglib>
    <taglib-uri>/tags/struts-nested</taglib-uri>
    <taglib-location>/WEB-INF/struts-nested.tld</taglib-location>
  </taglib>

  <taglib>
    <taglib-uri>/tags/struts-tiles</taglib-uri>
    <taglib-location>/WEB-INF/struts-tiles.tld</taglib-location>
  </taglib>

</web-app>












-------------------------------------------------------
/**
* Copyright(c) Fujinet Co., Ltd.
* All rights reserved. 
* 
* @(#)LoginAction.java 01-00 2023/06/20
* 
* Version 1.00
* Last_Update 2023/07/11
*/

package fjs.cs.action;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.commons.beanutils.BeanUtils;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.apache.struts.action.ActionMessage;
import org.apache.struts.action.ActionMessages;
import org.springframework.web.struts.ActionSupport;

import fjs.cs.common.Constants;
import fjs.cs.form.LoginForm;
import fjs.cs.model.MSTUSER;
import fjs.cs.service.LoginService;

/**
 * LoginAction
 * 
 * This action handles the login function
 * 
 * @version 1.00
 * @since 1.00
 * 
 */

public class LoginAction extends ActionSupport {

	/**
	 * Processes the login request from the user.
	 *
	 * @param mapping   An ActionMapping object that contains information about the mapping of the Action.
	 * @param form      An ActionForm object that holds the login request data.
	 * @param request   An HttpServletRequest object that contains information about the HTTP request.
	 * @param response  An HttpServletResponse object that contains information about the HTTP response.
	 * @return          An ActionForward object indicating the forward path for the request.
	 * @throws Exception    If any exception occurs during the execution of the method.
	 */

	public ActionForward execute( ActionMapping mapping, ActionForm form, HttpServletRequest request,
								  HttpServletResponse response) throws Exception {

		LoginForm loginForm = (LoginForm) form;
		ActionMessages errors = loginForm.validateForm(mapping, request);
		String forward = Constants.FORWARD_FAILURE;
		
		if (!errors.isEmpty()) {
			this.saveErrors(request, errors);
		} else {
			LoginService loginService = (LoginService) getWebApplicationContext().getBean(Constants.BEAN_LOGIN);

			MSTUSER user = new MSTUSER();
			BeanUtils.copyProperties(user, loginForm);
			MSTUSER loggedInUser = loginService.getLoggedInUser(user);

			if (loggedInUser != null) {
				request.getSession().setAttribute("user", loggedInUser);
				forward = Constants.FORWARD_SUCCESS;
			} else {
				errors.add("", new ActionMessage("errors.notExist"));
				this.saveErrors(request, errors);
			}
		}
		return mapping.findForward(forward);
	}
}



Common----------------------
/**
* Copyright(c) Fujinet Co., Ltd.
* All rights reserved. 
* 
* @(#)Constants.java 01-00 2023/06/20
* 
* Version 1.00
* Last_Update 2023/07/11
*/
package fjs.cs.common;

/**
 * Constants
 * 
 * @version 1.00
 * @since  1.00
 * @author Tam-nlt
 * 
 */
public class Constants {
	// Table MSTUSER
	public static final String TABLE_USER = "MSTUSER";
	// Represents the next action to be performed based on the success result
	public static final String FORWARD_SUCCESS = "success";
	// Represents the next action to be performed based on the failure result
	public static final String FORWARD_FAILURE = "failure";
	// Successful login condition
	public static final int LOGIN_SUCCESS = 1;
	// Mode export
	public static final int EXPORT_MODE = -1;

	// List bean name in Spring Context
	public static final String BEAN_LOGIN = "loginService";

	
	

}




Dao-----------------
/**
* Copyright(c) Fujinet Co., Ltd.
* All rights reserved. 
* 
* @(#)LoginDao.java 01-00 2023/06/20
* 
* Version 1.00
* Last_Update 2023/07/11
*/

package fjs.cs.dao;

import org.hibernate.Query;
import org.springframework.orm.hibernate3.support.HibernateDaoSupport;

import fjs.cs.common.Constants;
import fjs.cs.model.MSTUSER;

/**
 * LoginDao
 * 
 * This class provides data access methods for the MSTUSER table in the database.
 * 
 * @version 1.00
 * @since 1.00
 * 
 */

public class LoginDao extends HibernateDaoSupport {
	
	/**
	 * Get information about logged in user
	 * 
	 * @param user The MSTUSER object
	 * @return the user logged in
	 */
	
	public MSTUSER getLoggedInUser(MSTUSER user) {
	    MSTUSER currentUser = null;
	    try {
	        String loginQuery = "SELECT COUNT(*) FROM " + Constants.TABLE_USER +" WHERE USERID = :userID AND PASSWORD = :password AND DELETE_YMD IS NULL";
	        
	        Query countQuery = getSession().createQuery(loginQuery)
	            .setParameter("userID", user.getUserID())
	            .setParameter("password", user.getPassword());
	        
	        int loginResult = ((Number) countQuery.uniqueResult()).intValue();

	        if (loginResult == Constants.LOGIN_SUCCESS) {
	            String getUserQuery = "FROM " + Constants.TABLE_USER +" WHERE USERID = :userID AND DELETE_YMD IS NULL";
	            Query query = getSession().createQuery(getUserQuery).setParameter("userID", user.getUserID());
	            currentUser = (MSTUSER) query.uniqueResult();
	        }
	    } catch (Exception e) {
	        e.printStackTrace();
	    }
	    return currentUser;
	}
}



form--------------
/**
* Copyright(c) Fujinet Co., Ltd.
* All rights reserved. 
* 
* @(#)LoginForm.java 01-00 2023/06/20
* 
* Version 1.00
* Last_Update 2023/07/11
*/

package fjs.cs.form;

import javax.servlet.http.HttpServletRequest;

import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionMapping;
import org.apache.struts.action.ActionMessage;
import org.apache.struts.action.ActionMessages;

/**
 * LoginForm
 * 
 * @version 1.00
 * @since 1.00
 * @author Tam-nlt
 *
 */

public class LoginForm extends ActionForm {

	private static final long serialVersionUID = 1L;
	private String userID;
	private String password;
	private int psnCd;

	public int getPsnCd() {
		return psnCd;
	}

	public void setPsnCd(int psnCd) {
		this.psnCd = psnCd;
	}

	public String getUserID() {
		return userID;
	}

	public void setUserID(String userID) {
		this.userID = userID;
	}

	public String getPassword() {
		return password;
	}

	public void setPassword(String password) {
		this.password = password;
	}

	/**
	 * Validates the user credentials.
	 *
	 * @param mapping The ActionMapping object that contains the mapping information
	 *                between URL and Action.
	 * @param request The HttpServletRequest object that contains the HTTP request
	 *                information from the browser.
	 * @return The ActionMessages object that contains any validation errors.
	 */

	public ActionMessages validateForm(ActionMapping mapping, HttpServletRequest request) {
		ActionMessages errors = new ActionMessages();
		System.out.println(userID +"-"+ password);
		if (userID == null || userID.isEmpty()) {
			errors.add("", new ActionMessage("errors.userID"));
		} else if (password == null || password.isEmpty()) {
			errors.add("", new ActionMessage("errors.password"));
		}
		return errors;
	}
}


hibernate ---------------------
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
"http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">
<hibernate-mapping>
    <class name="fjs.cs.model.MSTUSER" table="MSTUSER">
        <id name="psnCd" type="int">
            <column name="PSN_CD" />
            <generator class="assigned" />
        </id>
        <property name="userID" type="string">
            <column name="USERID" length="50" />
        </property>
        <property name="password" type="string">
            <column name="PASSWORD" length="50" />
        </property>
        <property name="userName" type="string">
            <column name="USERNAME" length="50" />
        </property>
        <property name="deleteYMD" type="timestamp">
            <column name="DELETE_YMD" />
        </property>
        <property name="insertYMD" type="timestamp">
            <column name="INSERT_YMD" />
        </property>
        <property name="insertPSNCD" type="int">
            <column name="INSERT_PSN_CD" />
        </property>
        <property name="updateYMD" type="timestamp">
            <column name="UPDATE_YMD" />
        </property>
        <property name="updatePSNCD" type="int">
            <column name="UPDATE_PSN_CD" />
        </property>
    </class>
</hibernate-mapping>




-----------Model

/**
* Copyright(c) Fujinet Co., Ltd.
* All rights reserved. 
* 
* @(#)MSTUSER.java 01-00 2023/06/20
* 
* Version 1.00
* Last_Update 2023/07/11
*/

package fjs.cs.model;

import java.io.Serializable;
import java.sql.Timestamp;

/**
 * MSTUSER
 * 
 * Data transfer object for MSTUSER entity.
 * 
 * @version 1.00
 * @since 1.00
 * 
 */

public class MSTUSER implements Serializable {

	private static final long serialVersionUID = 1L;
	
	private int psnCd;
	private String userID;
	private String password;
	private String userName;
	private Timestamp deleteYMD;
	private Timestamp insertYMD;
	private int insertPSNCD;
	private Timestamp updateYMD;
	private int updatePSNCD;

	/**
	 * Default constructor for MSTUSER class.
	 */
	public MSTUSER() {
		super();
	}
	
	/**
	 * Constructs a new instance of MSTUSER with the specified parameters.
	 *
	 * @param userID       the user ID
	 * @param password     the password
	 * @param userName     the user name
	 * @param deleteYMD    the deletion date
	 * @param insertYMD    the insertion date
	 * @param insertPSNCD  the insertion process code
	 * @param updateYMD    the update date
	 * @param updatePSNCD  the update process code
	 */
	
	public MSTUSER(String userID, String password, String userName, Timestamp deleteYMD, Timestamp insertYMD,
				   int insertPSNCD, Timestamp updateYMD, int updatePSNCD) {
		super();
		this.userID = userID;
		this.password = password;
		this.userName = userName;
		this.deleteYMD = deleteYMD;
		this.insertYMD = insertYMD;
		this.insertPSNCD = insertPSNCD;
		this.updateYMD = updateYMD;
		this.updatePSNCD = updatePSNCD;
	}

	/*Getter and Setter*/
	
	public int getPsnCd() {
		return psnCd;
	}

	public void setPsnCd(int psnCd) {
		this.psnCd = psnCd;
	}

	public String getUserID() {
		return userID;
	}

	public void setUserID(String userID) {
		this.userID = userID;
	}

	public String getPassword() {
		return password;
	}

	public void setPassword(String password) {
		this.password = password;
	}

	public String getUserName() {
		return userName;
	}

	public void setUserName(String userName) {
		this.userName = userName;
	}

	public Timestamp getDeleteYMD() {
		return deleteYMD;
	}

	public void setDeleteYMD(Timestamp deleteYMD) {
		this.deleteYMD = deleteYMD;
	}

	public Timestamp getInsertYMD() {
		return insertYMD;
	}

	public void setInsertYMD(Timestamp insertYMD) {
		this.insertYMD = insertYMD;
	}

	public int getInsertPSNCD() {
		return insertPSNCD;
	}

	public void setInsertPSNCD(int insertPSNCD) {
		this.insertPSNCD = insertPSNCD;
	}

	public Timestamp getUpdateYMD() {
		return updateYMD;
	}

	public void setUpdateYMD(Timestamp updateYMD) {
		this.updateYMD = updateYMD;
	}

	public int getUpdatePSNCD() {
		return updatePSNCD;
	}

	public void setUpdatePSNCD(int updatePSNCD) {
		this.updatePSNCD = updatePSNCD;
	}
}



server-----------


/**
* Copyright(c) Fujinet Co., Ltd.
* All rights reserved. 
* 
* @(#)LoginService.java 01-00 2023/06/20
* 
* Version 1.00
* Last_Update 2023/07/11
*/

package fjs.cs.service;

import fjs.cs.dao.LoginDao;
import fjs.cs.model.MSTUSER;

/**
 * Class for handling user login operations.
 * 
 * @version 1.00
 * @since 1.00
 */

public class LoginService{
		
	private LoginDao loginDao;

	public void setLoginDao(LoginDao loginDao) {
		this.loginDao = loginDao;
	}
		
	/**
	 * Retrieves the user details based on the user mapping from LoginForm
	 *
	 * @param user The MSTUSER object
	 * @return A MSTUSER object containing the user details, or null if the user is not found.
	 */	
	
	public MSTUSER getLoggedInUser(MSTUSER user) {
		return loginDao.getLoggedInUser(user);
	}
}


-----------app.propertys


# -- Set information for the database connection --
db.serverName=127.0.0.1
db.dbName=customersystem
db.portNumber=1433
db.userID=admin
db.password=Abc12345
db.driverClassName=com.microsoft.sqlserver.jdbc.SQLServerDriver
db.url=jdbc:sqlserver://127.0.0.1:1433;instanceName=SQLEXPRESS;databaseName=customersystem


# -- Login --
login.title=Login Page
login.heading=Login Page!

# -- Search --
search.title=Search Page
search.heading=Search Page!

# -- Edit --
edit.title=Edit Page
edit.heading=Edit Page!

# -- Import --
import.title=Import Page
import.heading=Import Page!

# -- Setting Header --
setting.title=Setting Header Page
setting.heading=Setting Header Page!

# -- Validate login form --
errors.userID= \u30E6\u30FC\u30B6\u30FCID\u3092\u5165\u529B\u3057\u3066\u304F\u3060\u3055\u3044
errors.notExist = \u30E6\u30FC\u30B6\u30FCID\u307E\u305F\u306F\u30D1\u30B9\u30EF\u30FC\u30C9\u304C\u4E0D\u6B63\u3067\u3059\u3002
errors.password = \u30D1\u30B9\u30EF\u30FC\u30C9\u3092\u5165\u529B\u3057\u3066\u304F\u3060\u3055\u3044\u3002

-------applicationContext.xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:jdbc="http://www.springframework.org/schema/jdbc"
	xmlns:orm="http://www.springframework.org/schema/orm"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/jdbc
		http://www.springframework.org/schema/jdbc/spring-jdbc.xsd
		http://www.springframework.org/schema/orm
		http://www.springframework.org/schema/orm/spring-orm.xsd">
	
	<!-- Configure Data Source -->
 	<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="locations">
            <list>
                <value>WEB-INF/src/app.properties</value>
            </list>
        </property>
        <property name="ignoreResourceNotFound" value="true"/>
    </bean>
    
	<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
		<property name="driverClassName" value="${db.driverClassName}" />
		<property name="url" value="${db.url}" />
		<property name="username" value="${db.userID}" />
		<property name="password" value="${db.password}" />
	</bean>


	<!--Configure Hibernate SessionFactory -->
	<bean id="sessionFactory" class="org.springframework.orm.hibernate3.LocalSessionFactoryBean">
		<property name="dataSource" ref="dataSource" />
		
		<property name="mappingResources">
        <list>
            <value>fjs/cs/hibernate/MSTUSER.hbm.xml</value>
        </list>
    	</property>
    	
		<property name="hibernateProperties">
			<props>
				<prop key="hibernate.dialect">org.hibernate.dialect.SQLServerDialect</prop>
				<prop key="hibernate.show_sql">true</prop>
				<prop key="hibernate.format_sql">true</prop>
			</props>
		</property>
	</bean>


	<!-- Configure Transaction Manager -->
	<bean id="transactionManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
		<property name="sessionFactory" ref="sessionFactory" />
	</bean>


	<!-- Configure Struts 1 -->
	
	<!-- DAO Beans -->
	 <bean id="loginDao" class="fjs.cs.dao.LoginDao" >
		 <property name="sessionFactory" ref="sessionFactory" />
   	</bean>
	
	<!-- Service Beans -->
   	<bean id="loginService" class="fjs.cs.service.LoginService" >
   		<property name="loginDao" ref="loginDao" />
   	</bean>
</beans>                   



-------------------Struts-config.xml
<?xml version="1.0" encoding="ISO-8859-1" ?>

<!DOCTYPE struts-config PUBLIC
          "-//Apache Software Foundation//DTD Struts Configuration 1.2//EN"
          "http://jakarta.apache.org/struts/dtds/struts-config_1_2.dtd">

<struts-config>

	<!-- Form Bean Definitions -->
	<form-beans>
		<form-bean name="loginForm" type="fjs.cs.form.LoginForm" />
	</form-beans>
	
	<!-- Global Exception Definitions -->
	<global-exceptions>
		<!-- sample exception handler <exception key="expired.password" type="app.ExpiredPasswordException" 
			path="/changePassword.jsp"/> end sample -->
	</global-exceptions>


	<!-- Global Forward Definitions -->
	<global-forwards>
		<forward name="/Login" path="/Login.do" />
	</global-forwards>


	<!-- Action Mapping Definitions -->
	<action-mappings>
	    <action path="/Login" forward="/Login" />
			<action path="/Login" name="loginForm"  type="fjs.cs.action.LoginAction" validate="false">
				<forward name="failure" path="/Login"/>
				<forward name="success" path="/Search.do" redirect="true"/>
				<forward name="success" path="/pages/Search.jsp" redirect="true"/>
			</action>
	</action-mappings>
	
	<action-mappings>
	    <action path="/Search" forward="/Search" />
			<action path="/Search" name="customerForm"  type="fjs.cs.action.SearchAction" validate="false">
				<forward name="failure" path="/Search"/>
				<forward name="success" path="/Search.do" redirect="true"/>
			</action>
	</action-mappings>


	<!-- Controller Configuration -->
	<controller processorClass="org.apache.struts.tiles.TilesRequestProcessor" />

	<!-- Message Resources Definitions -->
	<message-resources parameter="app" />

	<!-- Plug Ins Configuration -->
	
	<plug-in className="org.springframework.web.struts.ContextLoaderPlugIn">
    	<set-property property="contextConfigLocation" value="/WEB-INF/applicationContext.xml" />
	</plug-in>
	
	<plug-in className="org.apache.struts.tiles.TilesPlugin">
		<set-property property="definitions-config" value="/WEB-INF/tiles-defs.xml" />
		<set-property property="moduleAware" value="true" />
	</plug-in>

	<plug-in className="org.apache.struts.validator.ValidatorPlugIn">
		<set-property property="pathnames" value="/WEB-INF/validator-rules.xml,/WEB-INF/validation.xml" />
	</plug-in>
</struts-config>




---------------web.xml

<?xml version="1.0" encoding="ISO-8859-1"?>

<!DOCTYPE web-app
  PUBLIC "-//Sun Microsystems, Inc.//DTD Web Application 2.2//EN"
  "http://java.sun.com/j2ee/dtds/web-app_2_2.dtd">

<web-app>
	<display-name>Struts Blank Application</display-name>

    
	<!-- Standard Action Servlet Configuration (with debugging) -->
	<servlet>
		<servlet-name>action</servlet-name>
		<servlet-class>org.apache.struts.action.ActionServlet</servlet-class>
		<init-param>
			<param-name>config</param-name>
			<param-value>/WEB-INF/struts-config.xml</param-value>
		</init-param>
		<init-param>
			<param-name>debug</param-name>
			<param-value>2</param-value>
		</init-param>
		<init-param>
			<param-name>detail</param-name>
			<param-value>2</param-value>
		</init-param>
		<load-on-startup>2</load-on-startup>
	</servlet>


	<!-- Standard Action Servlet Mapping -->
	<servlet-mapping>
		<servlet-name>action</servlet-name>
		<url-pattern>*.do</url-pattern>
	</servlet-mapping>

	<servlet>
		<servlet-name>T001Servlet</servlet-name>
		<jsp-file>/pages/Login.jsp</jsp-file>
	</servlet>
	<servlet-mapping>
		<servlet-name>T001Servlet</servlet-name>
		<url-pattern>/Login</url-pattern>
	</servlet-mapping>
	
	<context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/applicationContext.xml</param-value>
    </context-param>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>


	<!-- The Usual Welcome File List -->
	<welcome-file-list>
		<welcome-file>index.jsp</welcome-file>
	</welcome-file-list>


	<!-- Struts Tag Library Descriptors -->
	<taglib>
		<taglib-uri>/tags/struts-bean</taglib-uri>
		<taglib-location>/WEB-INF/struts-bean.tld</taglib-location>
	</taglib>

	<taglib>
		<taglib-uri>/tags/struts-html</taglib-uri>
		<taglib-location>/WEB-INF/struts-html.tld</taglib-location>
	</taglib>

	<taglib>
		<taglib-uri>/tags/struts-logic</taglib-uri>
		<taglib-location>/WEB-INF/struts-logic.tld</taglib-location>
	</taglib>

	<taglib>
		<taglib-uri>/tags/struts-nested</taglib-uri>
		<taglib-location>/WEB-INF/struts-nested.tld</taglib-location>
	</taglib>

	<taglib>
		<taglib-uri>/tags/struts-tiles</taglib-uri>
		<taglib-location>/WEB-INF/struts-tiles.tld</taglib-location>
	</taglib>
	    
   
</web-app>




---index.jsp


<%@ taglib uri="/tags/struts-logic" prefix="logic" %>
<logic:redirect forward="/Login"/>





---login


<%@ page language="java" contentType="text/html; charset=utf-8"%>
<%@include file="../WEB-INF/common/Taglib.jsp" %>
<html:html locale="true">
<head>
<title><bean:message key="login.title"/></title>
<style type="text/css">
	<%@include file="../WEB-INF/css/Login.css" %>
	<%@include file="../WEB-INF/css/Base.css" %>
</style>
<script src="./src/js/Login.js"></script>
<html:base/>
</head>
<body>
	<%@include file="../WEB-INF/common/Header.jsp" %>
	<div class="content">
		<div class="content-text">Login</div>
		<div class="content-login">
			   <form action="./Login.do" method="POST">
				<div class="content-login__header">
					<h3>LOGIN</h3>
					<div id="lblErrorMessage">
						 <html:errors/>
					</div>
				</div>
				<div class="content-login__container">
					<div class="form-group">
						<label for="fullname" class="form-label form-label__userID">User
							Id: </label>
						<input name="userID" id="txtUserID" value="" maxlength="8" class="form-control" />
					</div>
					<div class="form-group__password">
						<label for="password" class="form-label">Password: </label>
						<html:password property="password" styleId="txtPassword"  value='' maxlength="8" styleClass="form-control" />
					</div>
					<div class="form-group__btn">
						<html:submit styleId="btnLogin" styleClass="form-submit" value="Login" />
						<html:reset styleId="btnClear" styleClass="form-submit" value="Clear"/>
					</div>
				</div>
			</form>
		</div>
	</div>
	<%@include file="../WEB-INF/common/Footer.jsp" %>	
</body>
</html:html>





<param property="username" name="username" />













/**
* Copyright(c) Fujinet Co., Ltd.
* All rights reserved. 
* 
* @(#)LoginAction.java 01-00 2023/06/20
* 
* Version 1.00
* Last_Update 2023/07/11
*/

package fjs.cs.action;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.commons.beanutils.BeanUtils;
import org.apache.struts.action.ActionForm;
import org.apache.struts.action.ActionForward;
import org.apache.struts.action.ActionMapping;
import org.apache.struts.action.ActionMessage;
import org.apache.struts.action.ActionMessages;
import org.springframework.web.struts.ActionSupport;

import fjs.cs.common.Constants;
import fjs.cs.form.LoginForm;
import fjs.cs.model.MSTUSER;
import fjs.cs.service.LoginService;

/**
 * LoginAction
 * 
 * This action handles the login function
 * 
 * @version 1.00
 * @since 1.00
 * 
 */

public class LoginAction extends ActionSupport {

	/**
	 * Processes the login request from the user.
	 *
	 * @param mapping   An ActionMapping object that contains information about the mapping of the Action.
	 * @param form      An ActionForm object that holds the login request data.
	 * @param request   An HttpServletRequest object that contains information about the HTTP request.
	 * @param response  An HttpServletResponse object that contains information about the HTTP response.
	 * @return          An ActionForward object indicating the forward path for the request.
	 * @throws Exception    If any exception occurs during the execution of the method.
	 */

	public ActionForward execute( ActionMapping mapping, ActionForm form, HttpServletRequest request,
								  HttpServletResponse response) throws Exception {

		LoginForm loginForm = (LoginForm) form;
		ActionMessages errors = loginForm.validateForm(mapping, request);
		String forward = Constants.FORWARD_FAILURE;
		
		System.out.println(loginForm.getUsername() + "nguyenvana" + loginForm.getPassword());
		String userName = loginForm.getUsername();
		String passWord = loginForm.getPassword();
		if (userName != null && passWord != null) {
			LoginService loginService = (LoginService) getWebApplicationContext().getBean(Constants.BEAN_LOGIN);
			MSTUSER user = new MSTUSER();
			user.setUserID(userName);
			user.setPassword(passWord);
			MSTUSER loggedInUser = loginService.getLoggedInUser(user);
			if (loggedInUser != null) {
				request.getSession().setAttribute("user", loggedInUser);
				forward = Constants.FORWARD_SUCCESS;
			} else {
				errors.add("", new ActionMessage("errors.notExist"));
				this.saveErrors(request, errors);
			}
		}
		
		if (!errors.isEmpty()) {
			this.saveErrors(request, errors);
		} else {
			LoginService loginService = (LoginService) getWebApplicationContext().getBean(Constants.BEAN_LOGIN);

			MSTUSER user = new MSTUSER();
			BeanUtils.copyProperties(user, loginForm);
			MSTUSER loggedInUser = loginService.getLoggedInUser(user);

			if (loggedInUser != null) {
				request.getSession().setAttribute("user", loggedInUser);
				forward = Constants.FORWARD_SUCCESS;
			} else {
				errors.add("", new ActionMessage("errors.notExist"));
				this.saveErrors(request, errors);
			}
		}
		return mapping.findForward(forward);
	}
}



Database--------------------------
create database customersystem;
use customersystem;

create table MSTUSER (
	PSN_CD NUMERIC(4) primary key not null,
	USERID VARCHAR(8),
	PASSWORD VARCHAR(8),
	USERNAME VARCHAR(40),
	DELETE_YMD TIMESTAMP default(null),
	INSERT_YMD TIMESTAMP default(CURRENT_TIMESTAMP),
	INSERT_PSN_CD NUMERIC(5) default(0),
	UPDATE_YMD TIMESTAMP default(null),
	UPDATE_PSN_CD NUMERIC(5) default(5)
)

create table MSTCUSTOMER(
	
)





public List<Object[]> getData() {
    StringBuilder hql = new StringBuilder();
    hql.append("SELECT customerID, customerName, ");
    hql.append("CASE WHEN sex = 0 THEN 'Male' ELSE 'Female' END AS sex, ");
    hql.append("birthday, address ");
    hql.append("FROM MSTCustomer ");
    hql.append("WHERE deleteYMD IS NULL ");
    hql.append("ORDER BY customerID");

    Query query = getSession().createQuery(hql.toString());
    List<Object[]> results = query.list();

    return results;
}





public ActionForward execute( ActionMapping mapping, ActionForm form, HttpServletRequest request,
								  HttpServletResponse response) throws Exception {

		LoginForm loginForm = (LoginForm) form;
		ActionMessages errors = loginForm.validateForm(mapping, request);
		String forward = Constants.FORWARD_FAILURE;
	
		String userName = loginForm.getUsername();
		String passWord = loginForm.getPassword();
		if (userName != null && passWord != null) {
			LoginService loginService = (LoginService) getWebApplicationContext().getBean(Constants.BEAN_LOGIN);
			MSTUSER user = new MSTUSER();
			user.setUserID(userName);
			user.setPassword(passWord);
			MSTUSER loggedInUser = loginService.getLoggedInUser(user);
			if (loggedInUser != null) {
				request.getSession().setAttribute("user", loggedInUser);
				forward = Constants.FORWARD_SUCCESS;
			} else {
				errors.add("", new ActionMessage("errors.urlLogin" + userName + passWord));
				this.saveErrors(request, errors);
			}
		}
		
		if (!errors.isEmpty()) {
			this.saveErrors(request, errors);
		} else {
			LoginService loginService = (LoginService) getWebApplicationContext().getBean(Constants.BEAN_LOGIN);

			MSTUSER user = new MSTUSER();
			BeanUtils.copyProperties(user, loginForm);
			MSTUSER loggedInUser = loginService.getLoggedInUser(user);

			if (loggedInUser != null) {
				request.getSession().setAttribute("user", loggedInUser);
				forward = Constants.FORWARD_SUCCESS;
			} else {
				errors.add("", new ActionMessage("errors.notExist"));
				this.saveErrors(request, errors);
			}
		}
		return mapping.findForward(forward);
	}

















public ActionForward execute( ActionMapping mapping, ActionForm form, HttpServletRequest request, HttpServletResponse response) throws Exception {
    LoginForm loginForm = (LoginForm) form;
    ActionMessages errors = loginForm.validateForm(mapping, request);
    String forward = Constants.FORWARD_FAILURE;

    String userName = loginForm.getUsername();
    String passWord = loginForm.getPassword();

    if (userName != null && passWord != null) {
        forward = processLogin(userName, passWord, request, errors, loginForm);
    } else {
        forward = Constants.FORWARD_FAILURE;
        errors.add("", new ActionMessage("errors.urlLogin" + userName + passWord));
        this.saveErrors(request, errors);
    }

    return mapping.findForward(forward);
}

private String processLogin(String userName, String passWord, HttpServletRequest request, ActionMessages errors, LoginForm loginForm) {
    LoginService loginService = (LoginService) getWebApplicationContext().getBean(Constants.BEAN_LOGIN);
    MSTUSER user = new MSTUSER();
    user.setUserID(userName);
    user.setPassword(passWord);
    MSTUSER loggedInUser = loginService.getLoggedInUser(user);

    if (loggedInUser != null) {
        request.getSession().setAttribute("user", loggedInUser);
        return Constants.FORWARD_SUCCESS;
    } else {
        errors.add("", new ActionMessage("errors.urlLogin" + userName + passWord));
        this.saveErrors(request, errors);
        return Constants.FORWARD_FAILURE;
    }
}
