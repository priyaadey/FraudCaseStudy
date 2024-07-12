# FraudCaseStudy

## Team Name: Everest

## Team Members
1. Priya Dey
2. Ansley Williams
3. Jennifer Mai
4. Sowmya Sri Gupta
5. Sabina Tong

## Proposal:
Develop a case that addresses fraud risks in a corporate environment and leverages the use of information technology in optimizing fraud detection and mitigating risk. We cover specific instances of fraud (financial statement fraud, credit card fraud, and expense reimbursement fraud).

## Database Information

This is a mock database that can be viewed as either an extension of Workday's current database or Crowe's internal expense reimbursement database.


## Financial Fraud Detection 

### PROBLEM STATEMENT 
The problem at hand involves the manipulation of sales figures, overstatement of assets, accounts receivable, and concealment within financial statements. These fraudulent activities undermine the accuracy and integrity of financial reporting, posing significant risks to organizational  transparency and compliance. 
### RESEARCH INSIGHTS
Existing methods rely on predefined rules and human supervision, limiting effectiveness against evolving fraud schemes and analyzing large volumes of unstructured data.
### PROPOSED SOLUTION
Implementing Machine Learning (ML) algorithms to: 
- Adaptability: ML algorithms continuously learn from historical data to detect new fraud patterns without relying solely on predefined rules.
- Enhanced Analysis: AI capabilities extend to analyzing unstructured data formats like images and PDFs, significantly increasing the scope and depth of fraud detection.
- Scalability: Addressing the limitation of smaller data sample sizes with ML enables more comprehensive fraud analysis.


# Credit Card Fraud

### PROBLEM STATEMENT
Credit card fraud remains a significant concern due to vulnerabilities in data security and the adaptability of fraudsters to circumvent traditional detection methods.
### RESEARCH INSIGHTS
Companies face ongoing challenges in swiftly adapting to new fraud techniques, resulting in occasional lapses in detection effectiveness.
### PROPOSED SOLUTION
Leveraging Generative AI (GenAI) for enhanced pattern recognition and detection capabilities offers a proactive approach. ML algorithms trained on historical data minimize false positives, thereby reducing customer inconvenience and inaccurate fraud alerts. By combining diverse attributes such as location and purchase time intervals, ML models can identify nuanced fraudulent activities that evade traditional  rule-based systems.
### REBUTTAL TO CONCERNS
Acknowledging ML's inherent limitations in achieving 100% accuracy, deep neural networks are employed to optimize detection outcomes while mitigating customer impact. Despite fraudsters also leveraging AI, corporations possess superior access to comprehensive datasets, which when organized and leveraged effectively, enhance their detection capabilities.

# Expense Reimbursement Fraud

### PROBLEM STATEMENT
Instances of small-scale expense reimbursement fraud pose financial risks to organizations.
### RESEARCH INSIGHTS
Companies face ongoing challenges in swiftly adapting to new fraud techniques, resulting in occasional lapses in detection effectiveness.
### PROPOSED SOLUTION
Introducing CrowePAY, an app-based solution akin to Apple Pay, streamlines expense submission and verification processes.
### KEY FEATURES
- Direct Payment: Integration with corporate credit cards simplifies transactions, eliminating the need for manual receipt uploads.
- Automated Reporting: Selecting projects triggers automatic weekly expense reports, enhancing efficiency and accuracy.
- Verification Process: Utilizing Crowe's proprietary database, SQL codes cross-check receipt IDs to validate expense authenticity. An intuitive interface allows users and managers to verify, reject, or request additional information, ensuring thorough review and approval processes.




### 
In this code, the ExpenseCategory and ClientID are placeholders for the expense category and client ID from the receipt. The code calculates the minimum and maximum total amounts for unflagged receipts for the same type of purchase and client, and then checks if the total amount on the receipt is less than the minimum amount or more than 30% higher than the maximum amount. If so, it flags the receipt. 

-- Create the CompanyReceipts table 
CREATE TABLE CompanyReceipts ( 
    ReceiptID INT PRIMARY KEY, 
    TotalAmount DECIMAL(10, 2), 
    UserID INT, 
    Status VARCHAR(50) 
); 
 
-- Create the UserReceipts table 
CREATE TABLE UserReceipts ( 
    ReceiptID INT PRIMARY KEY, 
    TotalAmount DECIMAL(10, 2), 
    UserID INT 
); 
 
-- Insert some randomized values into the CompanyReceipts table 
INSERT INTO CompanyReceipts (ReceiptID, TotalAmount, UserID, Status) 
VALUES  
    (1, ROUND(RAND() * 100, 2), FLOOR(RAND() * 10) + 1, 'Unflagged'), 
    (2, ROUND(RAND() * 100, 2), FLOOR(RAND() * 10) + 1, 'Unflagged'), 
    (3, ROUND(RAND() * 100, 2), FLOOR(RAND() * 10) + 1, 'Unflagged'); 
 
-- Insert some randomized values into the UserReceipts table 
INSERT INTO UserReceipts (ReceiptID, TotalAmount, UserID) 
VALUES  
    (1, ROUND(RAND() * 100, 2), FLOOR(RAND() * 10) + 1), 
    (2, ROUND(RAND() * 100, 2), FLOOR(RAND() * 10) + 1), 
    (3, ROUND(RAND() * 100, 2), FLOOR(RAND() * 10) + 1); 

 

-- Checking receipt ID against out own DB to patch fraud risk areas 

UPDATE CompanyReceipts  
SET Status = CASE  

    -- Check if the receipt has already been submitted by the user 
    
    WHEN EXISTS (  
        SELECT 1  
        FROM UserReceipts U  
        WHERE U.ReceiptID = :ReceiptID AND U.ReceiptID = CompanyReceipts.ReceiptID  
    ) THEN 'Flagged'  
    
    -- Check if the total amount on the receipt has been altered
    
    WHEN EXISTS (  
        SELECT 1  
        FROM UserReceipts U  
        WHERE U.ReceiptID = :ReceiptID AND U.TotalAmount != :TotalAmount AND U.ReceiptID = CompanyReceipts.ReceiptID  
    ) THEN 'Flagged'  
    
    -- Check if multiple people are trying to submit the same receipt  
    
    WHEN EXISTS (  
        SELECT 1  
        FROM UserReceipts U  
        WHERE U.ReceiptID = :ReceiptID AND U.UserID != :UserID AND U.ReceiptID = CompanyReceipts.ReceiptID  
    ) THEN 'Flagged'  
    
     -- Check if the total amount is outside the range of amounts for similar receipt quantities within Crowe’s DB  

    WHEN EXISTS ( 
        SELECT 1 
        FROM ( 
            SELECT MIN(TotalAmount) as min_amount, MAX(TotalAmount) as max_amount 
            FROM CompanyReceipts 
            WHERE ExpenseCategory = :ExpenseCategory 
            AND ClientID = :ClientID 
            AND Status = 'Unflagged' 
            AND ReceiptID != :ReceiptID 
        ) B 
        WHERE :TotalAmount > 1.2 * B.max_amount 
    ) THEN 'Flagged' 
    
    -- If none of the above conditions are met, keep the current status  
    
    ELSE CompanyReceipts.Status  
END  
WHERE ReceiptID = :ReceiptID; 
