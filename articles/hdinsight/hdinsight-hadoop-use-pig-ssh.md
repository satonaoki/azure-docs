---
title: Use Hadoop Pig with SSH on an HDInsight cluster | Microsoft Docs
description: Learn how connect to a Linux-based Hadoop cluster with SSH, and then use the Pig command to run Pig Latin statements interactively, or as a batch job.
services: hdinsight
documentationcenter: ''
author: Blackmist
manager: jhubbard
editor: cgronlun
tags: azure-portal

ms.assetid: b646a93b-4c51-4ba4-84da-3275d9124ebe
ms.service: hdinsight
ms.custom: hdinsightactive
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 01/17/2017
ms.author: larryfr

---
# Run Pig jobs on a Linux-based cluster with the Pig command (SSH)
[!INCLUDE [pig-selector](../../includes/hdinsight-selector-use-pig.md)]

In this document you will walk through the process of connecting to a Linux-based Azure HDInsight cluster by using Secure Shell (SSH), then using the Pig command to run Pig Latin statements interactively, or as a batch job.

The Pig Latin programming language allows you to describe transformations that are applied to the input data to produce the desired output.

> [!NOTE]
> If you are already familiar with using Linux-based Hadoop servers, but are new to HDInsight, see [Linux-based HDInsight Tips](hdinsight-hadoop-linux-information.md).


## <a id="prereq"></a>Prerequisites
To complete the steps in this article, you will need the following.

* A Linux-based HDInsight (Hadoop on HDInsight) cluster.

  > [!IMPORTANT]
  > Linux is the only operating system used on HDInsight version 3.4 or greater. For more information, see [HDInsight Deprecation on Windows](hdinsight-component-versioning.md#hdi-version-33-nearing-deprecation-date).

* An SSH client. Linux, Unix, and Mac OS should come with an SSH client. Windows users must download a client, such as [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html).

## <a id="ssh"></a>Connect with SSH
Connect to the fully qualified domain name (FQDN) of your HDInsight cluster by using the SSH command. The FQDN will be the name you gave the cluster, then **.azurehdinsight.net**. For example, the following would connect to a cluster named **myhdinsight**.

    ssh admin@myhdinsight-ssh.azurehdinsight.net

**If you provided a certificate key for SSH authentication** when you created the HDInsight cluster, you may need to specify the location of the private key on your client system.

    ssh admin@myhdinsight-ssh.azurehdinsight.net -i ~/mykey.key

**If you provided a password for SSH authentication** when you created the HDInsight cluster, you will need to provide the password when prompted.

For more information on using SSH with HDInsight, see [Use SSH with HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md).

## <a id="pig"></a>Use the Pig command
1. Once connected, start the Pig command-line interface (CLI) by using the following command.

        pig

    After a moment, you should see a `grunt>` prompt.
2. Enter the following statement.

        LOGS = LOAD 'wasbs:///example/data/sample.log';

    This command loads the contents of the sample.log file into LOGS. You can view the contents of the file by using the following.

        DUMP LOGS;
3. Next, transform the data by applying a regular expression to extract only the logging level from each record by using the following.

        LEVELS = foreach LOGS generate REGEX_EXTRACT($0, '(TRACE|DEBUG|INFO|WARN|ERROR|FATAL)', 1)  as LOGLEVEL;

    You can use **DUMP** to view the data after the transformation. In this case, use `DUMP LEVELS;`.
4. Continue applying transformations by using the following statements. Use `DUMP` to view the result of the transformation after each step.

    <table>
    <tr>
    <th>Statement</th><th>What it does</th>
    </tr>
    <tr>
    <td>FILTEREDLEVELS = FILTER LEVELS by LOGLEVEL is not null;</td><td>Removes rows that contain a null value for the log level and stores the results into FILTEREDLEVELS.</td>
    </tr>
    <tr>
    <td>GROUPEDLEVELS = GROUP FILTEREDLEVELS by LOGLEVEL;</td><td>Groups the rows by log level and stores the results into GROUPEDLEVELS.</td>
    </tr>
    <tr>
    <td>FREQUENCIES = foreach GROUPEDLEVELS generate group as LOGLEVEL, COUNT(FILTEREDLEVELS.LOGLEVEL) as COUNT;</td><td>Creates a new set of data that contains each unique log level value and how many times it occurs. This is stored into FREQUENCIES.</td>
    </tr>
    <tr>
    <td>RESULT = order FREQUENCIES by COUNT desc;</td><td>Orders the log levels by count (descending) and stores into RESULT.</td>
    </tr>
    </table>
5. You can also save the results of a transformation by using the `STORE` statement. For example, the following saves the `RESULT` to the **/example/data/pigout** directory on the default storage container for your cluster.

        STORE RESULT into 'wasbs:///example/data/pigout';

   > [!NOTE]
   > The data is stored in the specified directory in files named **part-nnnnn**. If the directory already exists, you will receive an error.
   >
   >
6. To exit the grunt prompt, enter the following statement.

        QUIT;

### Pig Latin batch files
You can also use the Pig command to run Pig Latin contained in a file.

1. After exiting the grunt prompt, use the following command to pipe STDIN into a file named **pigbatch.pig**. This file will be created in the home directory for the account you are logged in to for the SSH session.

        cat > ~/pigbatch.pig
2. Type or paste the following lines, and then use Ctrl+D when finished.

        LOGS = LOAD 'wasbs:///example/data/sample.log';
        LEVELS = foreach LOGS generate REGEX_EXTRACT($0, '(TRACE|DEBUG|INFO|WARN|ERROR|FATAL)', 1)  as LOGLEVEL;
        FILTEREDLEVELS = FILTER LEVELS by LOGLEVEL is not null;
        GROUPEDLEVELS = GROUP FILTEREDLEVELS by LOGLEVEL;
        FREQUENCIES = foreach GROUPEDLEVELS generate group as LOGLEVEL, COUNT(FILTEREDLEVELS.LOGLEVEL) as COUNT;
        RESULT = order FREQUENCIES by COUNT desc;
        DUMP RESULT;
3. Use the following to run the **pigbatch.pig** file by using the Pig command.

        pig ~/pigbatch.pig

    Once the batch job finishes, you should see the following output, which should be the same as when you used `DUMP RESULT;` in the previous steps.

        (TRACE,816)
        (DEBUG,434)
        (INFO,96)
        (WARN,11)
        (ERROR,6)
        (FATAL,2)

## <a id="summary"></a>Summary
As you can see, the Pig command allows you to interactively run MapReduce operations by using Pig Latin, as well as run statements stored in a batch file.

## <a id="nextsteps"></a>Next steps
For general information on Pig in HDInsight.

* [Use Pig with Hadoop on HDInsight](hdinsight-use-pig.md)

For information on other ways you can work with Hadoop on HDInsight.

* [Use Hive with Hadoop on HDInsight](hdinsight-use-hive.md)
* [Use MapReduce with Hadoop on HDInsight](hdinsight-use-mapreduce.md)
