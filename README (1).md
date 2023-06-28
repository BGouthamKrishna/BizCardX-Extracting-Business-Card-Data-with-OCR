
# BizCardX: Extracting Business Card Data with OCR

BizCardX is a user-friendly tool for extracting information from business cards. The tool uses OCR technology to recognize text on business cards and extracts the data into a SQL database after classification using regular expressions. 

The extracted information include the company name, card holder
name, designation, mobile number, email address, website URL, area, city, state and pin code. The extracted information is then be displayed in the application's graphical user interface (GUI) called Streamlit. 

EasyOCR, as the name suggests, is a Python package that allows computer vision developers to effortlessly perform Optical Character Recognition. We use this package inorder optically recognise the text present on the business card. 


## Requirements 


Python3

MySQL

Streamlit

EasyOCR


## Installation

Install mysql and streamlit and mysql connector that connects mysql to Python

```bash
  Pip install streamlit 
  pip install mysql
  pip install mysql-connector-python
```
    
## Workflow

### Connect to the MySQL sever in order to access the database

```bash
  mydb = mysql.connector.connect(
    host="localhost",
    user="root",
    password="Goutham3",
    database="ocr")

mycursor = mydb.cursor()
```

### Create an empty table to store the business card information

```bash
  
mycursor.execute("CREATE TABLE IF NOT EXISTS bus (id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255), job_title VARCHAR(255), address VARCHAR(255), postcode VARCHAR(255), phone VARCHAR(255), email VARCHAR(255), website VARCHAR(255), company_name VARCHAR(225))")

```

### Create a sidebar menu with options to add, view, update, and delete business card information. User can choose what action he wants to perform

```bash
  
menu = ['Add', 'View', 'Update', 'Delete']
choice = st.sidebar.selectbox("Select an option", menu)

```

### Using easyocr to extract the text when the user seelects add option. 

```bash
  
uploaded_file = st.file_uploader("Upload a business card image", type=["jpg", "jpeg", "png"])
    if uploaded_file is not None:
        # Read the image using OpenCV
        image = cv2.imdecode(np.fromstring(uploaded_file.read(), np.uint8), 1)
        # Display the uploaded image
        st.image(image, caption='Uploaded business card image', use_column_width=True)
        # Create a button to extract information from the image
        if st.button('Extract Information'):
            reader = easyocr.Reader(['en'])
            results = reader.readtext(image)
            card_info = [i[1] for i in results]
            demilater = ' '
            card = demilater.join(card_info)  # convert to string
            replacement = [
                (";", ""),
                (',', ''),
                ("WWW ", "www."),
                ("www ", "www."),
                ('www', 'www.'),
                ('www.', 'www'),
                ('wwW', 'www'),
                ('wWW', 'www'),
                ('.com', 'com'),
                ('com', '.com'),

            ]
            for old, new in replacement:
                card = card.replace(old, new)

```

### Specifying the pattern for text extraciton in order to get results with higher accuracy rather than random text extraction 

```bash
  
# ----------------------Phone------------------------------------
            ph_pattern = r"\+*\d{2,3}-\d{3}-\d{4}"
            ph = re.findall(ph_pattern, card)
            Phone = ''
            for num in ph:
                Phone = Phone + ' ' + num
                card = card.replace(num, '')

            # ------------------Mail id--------------------------------------------
            mail_pattern = r"\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,3}\b"
            mail = re.findall(mail_pattern, card)
            Email_id = ''
            for ids in mail:
                Email_id = Email_id + ids
                card = card.replace(ids, '')

            # ---------------------------Website----------------------------------
            url_pattern = r"www\.[A-Za-z0-9]+\.[A-Za-z]{2,3}"
            url = re.findall(url_pattern, card)
            URL = ''
            for web in url:
                URL = URL + web
                card = card.replace(web, '')

            # ------------------------pincode-------------------------------------------
            pin_pattern = r'\d+'
            match = re.findall(pin_pattern, card)
            Pincode = ''
            for code in match:
                if len(code) == 6 or len(code) == 7:
                    Pincode = Pincode + code
                    card = card.replace(code, '')

            # ---------------name ,designation, company name-------------------------------
            name_pattern = r'^[A-Za-z]+ [A-Za-z]+$|^[A-Za-z]+$|^[A-Za-z]+ & [A-Za-z]+$'
            name_data = []  # empty list
            for i in card_info:
                if re.findall(name_pattern, i):
                    if i not in 'WWW':
                        name_data.append(i)
                        card = card.replace(i, '')
            name = name_data[0]
            designation = name_data[1]

            if len(name_data) == 3:
                company = name_data[2]
            else:
                company = name_data[2] + ' ' + name_data[3]
            card = card.replace(name, '')
            card = card.replace(designation, '')
            # city,state,address
            new = card.split()
            if new[4] == 'St':
                city = new[2]
            else:
                city = new[3]
            # state
            if new[4] == 'St':
                state = new[3]
            else:
                state = new[4]
            # address
            if new[4] == 'St':
                s = new[2]
                s1 = new[4]
                new[2] = s1
                new[4] = s  # swapping the St variable
                Address = new[0:3]
                Address = ' '.join(Address)  # list to string
            else:
                Address = new[0:3]
                Address = ' '.join(Address)  # list to string
            st.write('')
            st.write('###### :red[Name]         :', name)
            st.write('###### :red[Designation]  :', designation)
            st.write('###### :red[Company name] :', company)
            st.write('###### :red[Contact]      :', Phone)
            st.write('###### :red[Email id]     :', Email_id)
            st.write('###### :red[URL]          :', URL)
            st.write('###### :red[Address]      :', Address)
            st.write('###### :red[City]         :', city)
            st.write('###### :red[State]        :', state)
            st.write('###### :red[Pincode]      :', Pincode)

```

### Display the stored business card information under the view section

```bash
 mycursor.execute("SELECT * FROM bus")
    result = mycursor.fetchall()
    df = pd.DataFrame(result, columns=['id', 'name', 'job_title', 'address', 'postcode', 'phone', 'email', 'website',
                                       'company_name'])
    st.write(df)
```

### Create a dropdown menu which allows the user to select the card that they want to update under the undate option in the mainmenu

```bash
 # Create a dropdown menu to select a business card to update
    mycursor.execute("SELECT id, name FROM bus")
    result = mycursor.fetchall()
    business_cards = {}
    for row in result:
        business_cards[row[0]] = row[1]
    selected_card_id = st.selectbox("Select a business card to delete", business_cards)

    # Get the current information for the selected business card
    mycursor.execute("SELECT * FROM bus WHERE id=%s", (selected_card_id,))
    result = mycursor.fetchone()

    # Display the current information for the selected business card
    st.write("Name:", result[1])
    st.write("Job Title:", result[2])
    st.write("Address:", result[3])
    st.write("Postcode:", result[4])
    st.write("Phone:", result[5])
    st.write("Email:", result[6])
    st.write("Website:", result[7])
    st.write("company_name:", result[8])

    # Get new information for the business card
    name = st.text_input("Name", result[1])
    job_title = st.text_input("Job Title", result[2])
    address = st.text_input("Address", result[3])
    postcode = st.text_input("Postcode", result[4])
    phone = st.text_input("Phone", result[5])
    email = st.text_input("Email", result[6])
    website = st.text_input("Website", result[7])
    company_name = st.text_input("Company Name", result[8])

```

### Create a dropdown menu which allows the user to select the card that they want to delete under the delete option in the mainmenu

```bash
 
 mycursor.execute("SELECT id, name FROM bus")
    result = mycursor.fetchall()
    business_cards = {}
    for row in result:
        business_cards[row[0]] = row[1]
    selected_card_id = st.selectbox("Select a business card to delete", business_cards)

    # Get the name of the selected business card
    mycursor.execute("SELECT * FROM bus WHERE id=%s", (selected_card_id,))
    result = mycursor.fetchone()
   
    if st.button("Delete Business Card"):
        mycursor.execute("DELETE FROM bus WHERE id=%s", (selected_card_id,))
        mydb.commit()
        st.success("Business card information deleted from database.")

```


## Authors

- [@BGouthamKrishna](https://github.com/BGouthamKrishna)

