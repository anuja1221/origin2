# origin2
web-development
Validate_Passenger_Name()
{
    if [ ` echo $1|egrep "^[a-zA-Z]+[ ]?[a-zA-Z]+$" ` ]
    then
    return 1
    else
    echo "Invalid Name"
    exit 2
    fi
}
Validate_Passenger_Age()
{
      if [ ` echo $1|egrep "^[0-9]{2}$" ` ]
      then
      if [ $1 -ge 18 -a $1 -le 50 ]
      then
      return 1
      else
      echo "Age should be between 18 and 22"
      exit 3
      fi
      else
      echo "Invalid age"
      exit 3
      fi
}
Validate_Passenger_Phone()
{
      if [ `echo $1|egrep "^[6-9][0-9]{9}$" ` ]
      then
      return 1
      else
      echo "Invalid phone no"
      exit 4
      fi
}
Validate_Email_Id()
{
    if [ ` echo $1|egrep "^[a-zA-Z]+[0-9]*[._]?[a-zA-Z0-9]*[@][a-z]+[.][a-z]{2,4}$" ` ]
    then
    return 1
    else
    echo "Invalid Email Id"
    fi
}
Validate_Source_City()
{
      if [ ` echo $1|egrep "^[a-zA-Z]+$" ` ]
      then
      var=` egrep -i "$2" Buslist.txt|cut -d "#" -f4|egrep -i "$1"|wc -l`
      if [ $var -eq 1 ]
      then
      return 1
      else
      echo "Our Bus does not go there"
      fi
      else
      echo "Invalid Source City"
      fi
}
Validate_Destination_City()
{
    if [ ` echo $1|egrep "^[a-zA-Z]+$" ` ]
    then
    var=` egrep -i "$2" Buslist.txt|cut -d "#" -f5|egrep -i "$1"|wc -l`
    if [ $var -eq 1 ]
    then
    return 1
    else
    echo "Our Bus does not go there"
    exit 6
    fi
    else
    echo "Invalid Destination City"
    exit 6
    fi
}
Validate_Travel_Date()
{
    if [ ` echo $1|egrep "^[0-9]{2}-[0-9]{2}-[0-9]{4}$"` ]
    then
    v1=`  date +"%d-%m-%Y"|cut -d "-" -f3 `
    v2=`echo $1|cut -d "-" -f3 `
    if [ $v1 -lt $v2 ]
    then
    return 1
    elif [ $v1 -eq $v2 ]
    then
    v01=`  date +"%d-%m-%Y"|cut -d "-" -f2 `
    v02=`echo $1|cut -d "-" -f2 `
    if [ $v01 -lt $v02 ]
    then
    return 1
    elif [ $v01 -eq $v02 ]
    then
    v001=`  date +"%d-%m-%Y"|cut -d "-" -f1 `
    v002=`echo $1|cut -d "-" -f1 `
    if [ $v001 -le $v002 ]
    then
    return 1
    else
    echo   "Invalid date of trvelling"
    exit 7
    fi
    else
    echo "Invalid date of trvelling"
    exit 8
    fi
    else
    echo "Invalid date of travelling"
    exit 9
    fi
    else
    echo "Invalid Time"
    exit 10
    fi
}
auto_generate()
{
    if [ -e ticket.txt ]
    then
    tid=`tail -1 ticket.txt|cut -d ":" -f1|cut -c2-5 `
    id=`expr $tid + 1 `
    if [ $id -eq 1 ]
    then
    return 101
    else
    return $id
    fi
    else
    touch ticket.txt
    return 101
    fi
}

Book_Ticket()
{
    tr=$8
    Validate_Passenger_Name $1
    Validate_Passenger_Age $2
    Validate_Passenger_Phone $3
    Validate_Email_Id $4
    Validate_Source_City $5 $9
    Validate_Destination_City $6 $9
    Validate_Travel_Date $7
    ta=` egrep -i "$9" Buslist.txt|cut -d "#" -f2 `
    if [ $ta -ge $tr ]
    then
    auto_generate
    Tid=$?
    price=`egrep -i "$9" Buslist.txt|cut -d "#" -f3 `
    bill=` expr $price \* $tr `
    echo "T$Tid:$9:$1:$2:$3:$4:$5:$6:$7:$8:$bill" >>ticket.txt
    echo "Your Ticket Has Been Booked Succesfully"
    sr=` expr $ta - $tr `
    touch temp.txt
    while read line
    do
    if [ `echo $line|cut -d "#" -f1|egrep -i "$9" ` ]
    then
    echo "$9#$sr#$price#$5#$6" >>temp.txt
    else
    echo $line >>temp.txt
    fi
    done<Buslist.txt
    mv temp.txt Buslist.txt
    else
    echo "Ticket is Not available"
    fi
    Display
}
Cancel()
{
    eid=$1
    bname=$2
    flag=0
    ck=` egrep -i "$2" ticket.txt|cut -d ":" -f6 |egrep -i "$eid"|wc -l `
    if [ $ck -eq 1 ]
    then
    touch temp1.txt
    while read line
    do
    id=` echo $line|cut -d ":" -f6 `
    if [ $id == $eid ]
    then
    nroom=` echo $line|cut -d ":" -f10 `
    echo "Booking Canceled Successfully"
    flag=1
    else
    echo $line >>temp1.txt
    fi
    done <ticket.txt
    mv temp1.txt ticket.txt
    else
    echo "This email id Doesn't exist"
    fi
    if [ $flag -eq 1 ]
    then
    touch temp2.txt
    while read line
    do
    bn=` echo $line|cut -d "#" -f1 `
    echo $bn
    if [ $bn == $bname ]
    then
    var2=` echo $line|cut -d "#" -f2 `
    echo $var2
    var3=` echo $line|cut -d "#" -f3 `
    var4=` echo $line|cut -d "#" -f4 `
    var5=` echo $line|cut -d "#" -f5 `
    var6=` expr $var2 + $nroom `
    echo $var6
    echo "$bn#$var6#$var3#$var4#$var5" >>temp2.txt
    else
    echo "$line" >>temp2.txt
    fi
    done <Buslist.txt
    mv temp2.txt Buslist.txt
    fi
    }
    View_booking()
    {
    bname=$2
    id=` egrep -i "$bname" ticket.txt|cut -d ":" -f6|egrep -i "$1" `
    id2=` egrep -i "$bname" ticket.txt|cut -d ":" -f6|egrep -i "$1"|wc -l `
    if [ $id2 -eq 1 ]
    then
    while read line
    do
    id1=` echo $line|cut -d ":" -f6 `
    if [ $id1 == $id ]
    then
    echo "$line"
    fi
    done <ticket.txt
    else
    echo "This Email id Doesn't Exist"
    fi
}
menu()
      {
      echo "-----------------------Welcome To $2------------------------------"
      case $1 in
      1)
      #echo "-----------------------Welcome To $2------------------------------"
      bname=$2
      echo $bname
      read -p "Enter Your name" pname
      read -p "Enter Your Age" page
      read -p "Enter Your Phone No" phone
      read -p "Enter Your Email ID" peid
      read -p "Enter Your Source City" psrc
      read -p "Enter your destination City" pdes
      read -p "Enter date of travelling" pdate
      read -p "Enter Number Of tickets" pticket
      Book_Ticket "$pname" $page $phone $peid $psrc $pdes $pdate $pticket $bname
      ;;
      2)read -p "Enter Your Email id " eid
      Cancel $eid $2
      ;;
      3)read -p "Enter Your email id" eid
      View_booking $eid $2
      ;;
      4)Update
      ;;
      5)exit 1
      ;;
      *)echo "Enter Some Valid Choice"
      menu
      esac
}


Display()
{
      echo "-------------------------------Welcome $1----------------------------------"
      read -p "Which Bus You want to Book(FlyBus,SRS,Kaveri Travels)" bus
      typ=`cut -d "#" -f1 Buslist.txt|egrep -i "$bus"|wc -l`
      if [ $typ -eq 1 ]
      then
      flag=1
      else
      flag=0
      echo "You have entered Invalid Bus name"
      fi
      if [ $flag -eq 1 ]
      then
      echo -e "1.Book Ticket\n2.Cancel\n3.View_booking\n4.Update\n5.Exit"
      read -p "What do you want to do" ch1
      menu $ch1 $bus
      fi
}
Login()
{
      echo "************Welcome to login page***************"
      read -p "Enter Your Name" uname
      var=`echo "$uname"|egrep "^[a-zA-Z]+[ ]?[a-zA-Z]+$"|wc -l`
      if [ $var -eq 1 ]
      then
      echo "$uname" >>username.txt
      Display "$uname"
      else
      echo "Invalid Username"
      read -p "Do You want to Continue(Y/N)" ch
      if [ $ch == "Y" -o $ch == "y" ]
      then
      Login
      elif [ $ch == "N" -o $ch == "n" ]
      then
      exit 0
      else
      echo "Invalid entry"
      fi
      fi
}
Login
