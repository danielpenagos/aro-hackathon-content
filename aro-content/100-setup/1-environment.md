## The Workshop Environment You Are Using

!!! info "This workshop is best experienced in person with the Red Hat team guiding you through the content in a hosted workshop account. If you have stumbled across this workshop and wish to attempt it solo, we recommend creating a cluster via [this quickstart](https://mobb.ninja/docs/quickstart-aro/){:target='_blank'} and then skipping to the **Access Your Cluster** section of the workshop."

Your workshop environment consists of several components which have been pre-configured and are ready to use. This includes a [Microsoft Azure](https://azure.microsoft.com/en-us/){:target="_blank"} account, an [Azure Red Hat OpenShift](https://azure.microsoft.com/en-us/products/openshift/){:target="_blank"} cluster, and many other supporting resources.

To access your working environment, you'll need to log into the [Microsoft Azure portal](https://portal.azure.com){:target="_blank"}.  Use your provided [username](https://docs.google.com/spreadsheets/d/1A6LBraxWhtEH6zb1KCiiuerxgT8ATwsasBCvqxy1DgI/edit?usp=sharing). If you use the Azure portal regularly and it already knows a set of credentials for you, it's probably a good idea to use an Incognito window for this task, it should obviate the need to log out of an existing session.

When prompted, you'll log in with the credentials provided by the workshop team.

!!! warning "Log out of existing Microsoft Azure sessions"

    While these commands can be run in any Microsoft Azure account, we've completed many of the prerequisites for you to ensure they work in the workshop environment. As such, we recommend ensuring that you are logged out of any other Microsoft Azure sessions.


### Access Azure Cloud Shell

Azure Cloud Shell is an interactive, authenticated, browser-accessible shell for managing Azure resources. In this workshop, we'll use Azure Cloud Shell extensively to execute commands.

1. First, go ahead and skip the tour of the Azure Portal by clicking the *Maybe Later* button.

    ![Azure Portal Skip Tour](../assets/images/overview-skip-tour.png){ align=center }

1. To start Azure Cloud Shell, click on the `>_` button at the top right corner of the Azure Portal.

    ![Azure Portal Cloud Shell](../assets/images/overview-cloud-shell-icon.png){ align=center }

1. Once prompted, select *Bash* from the *Welcome* screen.

    ![Cloud Shell Language Choice](../assets/images/cloud-shell-bash.png){ align=center }

1. On the next screen, you'll receive a message that says "You have no storage mounted". Select the *Create storage* option.

    ![Cloud Shell Show Advanced Options](../assets/images/cloudshell-createstorage.png){ align=center }


1. When your shell is ready and you are at the bash prompt, run the following command to prepare your Cloud Shell environment for the remainder of the workshop:

    ```bash
    curl https://raw.githubusercontent.com/danielpenagos/aro-hackathon-content/gh-pages/assets/cloudshell-setup.sh > setup.sh
    chmod 755 setup.sh
    ./setup.sh
    source ~/.workshoprc
    ```

    You will see a significant amount of output as the script prepares your environment for the workshop.

    Congratulations, your Azure Cloud Shell is now configured and you're ready to move on to the next page.

> Note: If you are familiar with tmux and its shortcuts, it would be a good idea to run `tmux` now so that if the Azure Cloud Console times out you can easily connect back to the tmux and retain your working session.
