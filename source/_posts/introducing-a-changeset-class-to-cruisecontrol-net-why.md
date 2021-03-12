---
title: 'introducing a ChangeSet class to CruiseControl.net: why?'
id: 19
date: 2009-07-07 09:20:51
updated: 2009-07-07 09:20:51
tags:
- CruiseControl.net
- 'c#'
- introducing ChangeSet
- refactoring
---

I want to make changes to CruiseControl.net's object model to introduce the concept of [Changesets](http://mercurial.selenic.com/wiki/ChangeSet). This is going to be a large change, so I'm expecting I'll make a few more posts about it. My colleague [Mark Needham](http://www.markhneedham.com/blog/) already posted about a dojo where we did [some experimentation](http://www.markhneedham.com/blog/2009/06/12/coding-dojo-17-refactoring-cruise-control-net/) around this.

One of the important concepts in CruiseControl.net's domain is changes to source code. Right now, those changes are modeled with a Modification class:

<pre lang="csharp">///
/// Value object representing the data associated with a source control modification.
///
[XmlRoot("modification")]
public class Modification : IComparable
{
    public string Type = "unknown";
    public string FileName;
    public string FolderName;
    public DateTime ModifiedTime;
    public string UserName;
    public string ChangeNumber;
    public string Version = string.Empty;
    public string Comment;
    public string Url;
    public string IssueUrl;
    public string EmailAddress;
</pre>

Modification was part of the [first commit](http://bitbucket.org/davcamer/ccnet/changeset/e1e92ee1a612/) to sourceforge. Even back then, CruiseControl.net supported [several source control](http://bitbucket.org/davcamer/ccnet/src/457be6a966b7/project/core/sourcecontrol/) types: file-based, Perforce, PVCS, StarTeam and Visual Source Safe. A Modification object represents changes to one file. StarTeam, Perforce and PVCS have the concept of atomic changesets, but this is not supported by the model.

Changesets, though, [are important](http://www.catb.org/esr/writings/version-control/version-control.html#files_vs_filesets). All new SCM systems deal primarily with changesets, rather than single file modifications. This has been the dominant model since at least the year 2000 when Subversion was released. Because they are important, I would like to change ccnet to support them as first-class members of the model. Otherwise, there ends up being code like the following, which is primarily concerned with reconstructing changesets for display purposes:

<pre lang="csharp">private string WriteModificationsSummary(IEnumerable modifications)
{
    const string modificationHeaderFormat = "<tr><td>{0}</td><td>{1}</td></tr>";
    const string issueLinkFormat = "<tr><td>IssueLink</td><td><a>{0}</a></td></tr>";
    StringWriter mods = new StringWriter();

    mods.WriteLine("

#### Modifications in build :
");
    mods.WriteLine("<table>");
    ArrayList alreadyAdded = new ArrayList();
    foreach (Modification modification in modifications)
    {
        string modificationChecksum = modification.UserName + "__CCNET__" + modification.Comment;

        if (!alreadyAdded.Contains(modificationChecksum))
        {
            alreadyAdded.Add(modificationChecksum);

            mods.WriteLine(string.Format(modificationHeaderFormat,
                                         modification.UserName,
                                         modification.Comment));

            if (!string.IsNullOrEmpty(modification.IssueUrl))
            {
                mods.WriteLine(string.Format(issueLinkFormat,
                                             modification.IssueUrl));
            }
        }
    }
    mods.WriteLine("</table>");

    return mods.ToString();
}
</pre>

I know that I want to introduce a Changeset class. This Changeset will represent the changesets from SCMs that have the concept. For SCMs that don't it can represent groups of changes with the same comment and committer name, much like in the code above. In the long term, it should replace the Modification class.

At this point in any refactoring, I try to assess how much impact the refactoring will have. The way I go about the refactoring will be different if a lot of files will be touched, compared to one where only a few files will be touched. This is where ReSharper's Find Usage functionality really shines:

![screen-capture.png](http://intwoplacesatonce.com/wp-content/uploads/2009/07/screen-capture.png)

859 usages! So this is going to be a big refactoring. Even considering test code separately from application code: 291 in the application code, the remaining 563 in test code. There are 5 unlisted usages. I'm not sure why.

The "shape" of the listing is important as well as the size. The vast majority of the usages are in the Sourcecontrol namespace. I was expecting a lot of changes in the Sourcecontrol area because those classes will be returning a different type of object. That is the point of the refactoring. I was hoping for minimal impact on the rest of the ccnet code where Modification objects are consumed. Even in the long term, we can use an adapter sort of approach to work with Sourcecontrol blocks that can't be updated to directly produce Changesets. But in the rest of the application, it would be nice to completely remove Modification in a short to medium time frame. Only relatively few places in Publishers, Tasks and Core will need to be changed to accomplish that. So, even though there are many references to the Modification class they are in a good place for this change to work.

The only other thing of note on the listing is that a capitilization difference, "Sourcecontrol" versus "SourceControl", is causing the test's namespace to be split. At least that's a quick fix.
