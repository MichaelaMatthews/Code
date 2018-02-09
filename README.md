# Code
EXPRESSION ATLAS:

#Add to the db via Nayams code:
INSERT INTO exp_atlas (tissue,RT,Sample_no,RT_no_found) VALUES ('heart','A,B,C,D','10','4')

#Update the values for total exp_atlas rows in counts table:
UPDATE exp_atlas_count
SET counts = (SELECT COUNT(tissue) FROM exp_atlas);

#add or update the other counts for each tissue type:
#if the column exists, update it:
ALTER TABLE exp_atlas
	ADD heart INT(225) NOT NULL;
#else:
UPDATE exp_atlas_count
SET heart = (SELECT COUNT(tissue) FROM exp_atlas WHERE `tissue`="heart");

# View the overall % for tissues/total exp_atlas rows:
SELECT tissue, COUNT(tissue) AS RTs_no_found, GROUP_CONCAT(DISTINCT RT SEPARATOR ',') AS RT_found,concat(round(((SELECT COUNT(tissue))/(SELECT counts from exp_atlas_count)* 100 )),'%') FROM exp_atlas GROUP BY Tissue

#View the % for each family in a chosen tissue type:
SELECT tissue, COUNT(tissue) AS RTs_no_found, GROUP_CONCAT(DISTINCT RT SEPARATOR ',') AS RT_found,concat(round(((SELECT COUNT(RT))/(SELECT heart from exp_atlas_count)* 100 )),'%') FROM exp_atlas WHERE `tissue`='heart' GROUP BY RT


-------------- updating using PROCEDURE:
DELIMITER //
CREATE PROCEDURE UpdatecolumnIFExists()
BEGIN 
IF EXISTS(SELECT heart FROM exp_atlas_count)
THEN 
    UPDATE exp_atlas_count
    SET heart = (SELECT COUNT(tissue) FROM exp_atlas WHERE `tissue`='heart');
END IF;
END;
//

call UpdatecolumnIFExists();
------------------------------------
DELIMITER //
CREATE PROCEDURE AddcolumnUnlessExists()
BEGIN 
IF NOT EXISTS(SELECT heart FROM exp_atlas_count)
THEN 
    ALTER TABLE exp_atlas
	ADD heart INT(225) NOT NULL;
END IF;
END;
//

call AddcolumnUnlessExists();

-------drop procedures: 
drop procedure AddColumnUnlessExists
---------- show status:
SHOW PROCEDURE STATUS;
